```
pipeline :token_api do
  plug :accepts, ["json"]
  plug :fetch_session
  # NO TO  plug :protect_from_forgery
  plug :put_secure_browser_headers

  plug :verify_current_user_by_session
end

scope "/token" do
  pipe_through :token_api

  post "", TokenController, :create
end

defmodule TaApp.Web.Guest.TokenController do
  @moduledoc false
  use TaApp.Web, :controller

  alias TaApp.Web.Conn.AuthHelper

  plug :verify_current_user_by_token

  def create(conn, _params) do
    user = conn.assigns[:current_user]
    token = generate_signed_jwt(user)

    conn
    |> put_status(:created)
    |> render("create.json", %{token: token})
  end
end


@doc """
A plug that verifies the current user via the `Authorization` bearer token.
"""
def verify_current_user_by_token(conn, _opts \\ []) do
  case AuthHelper.get_user_by_token(conn) do
    {:ok, %{partner_id: ^partner_id} = user} ->
      Conn.assign(conn, :current_user, user)

    {:ok, _} ->
      ResponseHelper.unauthorized(conn, "Invalid user")

    {:error, _} ->
      ResponseHelper.unauthorized(conn, "Invalid user")
  end
end

def get_user_by_token(conn) do
  with {:ok, authorization} <- get_authorization(conn),
       {:ok, token} <- get_authorization_token(authorization),
       {:ok, user_id} <- verify_signed_jwt(token),
       {:ok, user} <- UserDomain.find(:id, user_id) do
    {:ok, user}
  else
    {:error, message} -> {:error, message}
    _ -> {:error, "not found"}
  end
end


alias Joken


@doc """
Generates a fully-signed JSON Web Token (JWT) for a particular user for use by
front end clients. The token is set to expire within 15 minutes from generation time.

Use the `generate_signed_jwt/1` function to generate a fully-signed
token in binary format.
"""
def generate_signed_jwt(user) do
  %Joken.Token{}
  |> Joken.with_json_module(Poison)
  |> Joken.with_exp(Joken.current_time() + 15 * 60)
  |> Joken.with_iat(Joken.current_time())
  |> Joken.with_nbf(Joken.current_time() - 1)
  |> Joken.with_sub(to_string(user.id))
  |> Joken.with_signer(Joken.hs256(jwt_secret()))
  |> Joken.sign()
  |> Joken.get_compact()
end



@doc """
Verifies the signed token and fetches the user record from the database
if the token is valid. Otherwise, returns an error.

## Examples

    get_user_by_token(conn)
    => %{:ok, user}

    get_user_by_token(conn)
    => %{:error, "the error message goes here"}
"""
def get_user_by_token(conn) do
  with {:ok, authorization} <- get_authorization(conn),
       {:ok, token} <- get_authorization_token(authorization),
       {:ok, user_id} <- verify_signed_jwt(token),
       {:ok, user} <- UserDomain.find(:id, user_id) do
    {:ok, user}
  else
    {:error, message} -> {:error, message}
    _ -> {:error, "not found"}
  end
end

@doc """
Verifies a signed JSON Web Token (JWT).
"""
def verify_signed_jwt(signed_token) do
  verified =
    signed_token
    |> Joken.token()
    |> Joken.with_signer(Joken.hs256(jwt_secret()))
    |> Joken.with_validation("exp", &(&1 > Joken.current_time()), "Token expired")
    |> Joken.with_validation("iat", &(&1 <= Joken.current_time()))
    |> Joken.with_validation("nbf", &(&1 < Joken.current_time()))
    |> Joken.verify()

  case verified do
    %Joken.Token{claims: %{"sub" => user_id}, error: nil} ->
      {:ok, user_id}

    %Joken.Token{error: error} ->
      {:error, error}
  end
end

defp jwt_secret do
  Application.get_env(:ta_app, TaApp.Web.Endpoint)[:secret_key_base]
end



assign_user_token()
conn
|> assign(:api_token, Auth.generate_signed_jwt(user))

<html lang="en">
  <head>

    <%= if assigns[:api_token] do %>
      <meta name="token" content="<%= @api_token %>">
    <% end %>


let apiToken = document.head.querySelector("meta[name='api_token']").content;

REFRESH THE TOKEN WHEN REQUIRED
```