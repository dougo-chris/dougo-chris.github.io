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
