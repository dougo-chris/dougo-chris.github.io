---
title: Underscore Keys
has_more: true
tags:
  - Javascript
  - Params
---

My elixir norms are all params are snakecase, yet in all my javascript code I use camelcase. Here's how I ensure any params are in the right format

<!--more-->

#### Create a plug to convert the keys
```
  def underscore_keys(conn, _opts \\ []) do
    params = ParamsHelper.underscore_keys(conn.params)
    %{conn | params: params}
  end
```

#### Convert all keys to camelcase
```
defmodule ParamsHelper do

  def underscore_keys(nil), do: nil

  def underscore_keys(%{} = map) do
    map
    |> Enum.map(fn {k, v} -> {Macro.underscore(k), underscore_keys(v)} end)
    |> Enum.into(%{})
  end

  def underscore_keys([head | rest]) do
    [underscore_keys(head) | underscore_keys(rest)]
  end

  def underscore_keys(not_a_map) do
    not_a_map
  end

```
