Mix.env/0 works correctly in mix phoenix.server, but it fails to call in a production environment which is built with exrm. It makes sense because mix isn't included in the release build, but is there any equivalent of Mix.env/0?

(UndefinedFunctionError) undefined function Mix.env/0 (module Mix is not available)
I'm using Mix.env/0 like this in some code:

if Mix.env == :dev do
  # xxxxxx
else
  # xxxxxx
end


config/config.exs:
config :my_app, :environment, Mix.env()

if Application.get_env(:my_app, :environment) == :dev do