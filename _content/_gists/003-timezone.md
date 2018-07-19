---
title: Timezone
has_more: true
tags:
  - Phoenix
  - Date & Time
---

Dealing with dates, times and timezones can be a real pain.
I allow users of my apps to set their timezone and I store all times as GMT.
Then with a few simple steps I adjust all dates and times to the right timezone

<!--more-->

### Create a plug to get the timezone
```
def assign_timezone(conn, _opts \\ []) do
  user = conn.assigns[:current_user]

  tz =
    case user do
      %{time_zone: time_zone} -> time_zone
      _ -> "UTC"
    end

  Conn.assign(conn, :tz, tz)
end
```

### The timezone is now in all views
```
  def render("show.json", %{user: user, tz: tz}) do
    %{
      name: user.name,
      insertedAt: shift_tz!(user.inserted_at, tz),
    }
  end
```

### Adjust the times
```
  defp shift_tz!(_, nil), do: nil

  defp shift_tz!(_, ""), do: nil

  defp shift_tz!(timestamp, tz) do
    case Timezone.convert(timestamp, tz) do
      {:error, _} -> timestamp
      timestamp -> timestamp
    end
  end

```
