---
title: Convert string parameters to atoms
has_more: true
tags:
  - Elixir
  - Phoenix
  - Params
---

It's unsafe to convert any string to an atom. I ensure only known keys are converted to atoms with this code

<!--more-->

#### Convert only known keys
```
@user_params ["full_name", "email", "password", "password_confirmation"]

def create(conn, params) do
  key_to_atom(params, @user_params)
  ...
  conn
end

def key_to_atom(map, allowed) do
  map
  |> Map.take(allowed)
  |> Enum.reduce(%{}, fn {key, value}, acc ->
    Map.put(acc, String.to_atom(key), value)
  end)
end

```
