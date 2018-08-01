---
title: Timestamp
has_more: true
tags:
  - Elixir
  - Erlang
  - Date & Time
---

Convert a timestamp into a string

<!--more-->

```
def create_timestamp do
  { {year, month, day}, {hour, minute, second} } = :calendar.now_to_universal_time(:os.timestamp())
  timestamp = :io_lib.format("~4..0B~2..0B~2..0B~2..0B~2..0B~2..0B", [year, month, day, hour, minute, second])
  timestamp |> List.flatten |> to_string
end
```

From (https://github.com/cogini)[https://github.com/cogini/elixir-deploy-template/blob/master/lib/mix/tasks/deploy.ex]

### another option
```
  {:ok, dt} =
    :seconds
    |> :os.system_time()
    |> DateTime.from_unix()

  [
    dt.year |> Integer.to_string() |> String.pad_leading(4, "0"),
    dt.month |> Integer.to_string() |> String.pad_leading(2, "0"),
    dt.day |> Integer.to_string() |> String.pad_leading(2, "0"),
    dt.hour |> Integer.to_string() |> String.pad_leading(2, "0"),
    dt.minute |> Integer.to_string() |> String.pad_leading(2, "0"),
    dt.second |> Integer.to_string() |> String.pad_leading(2, "0")
  ]
  |> Enum.join("-")
```