---
title:  Mix.env in production
tags:
  - Phoenix
  - Release
---

Mix.env/0 works correctly in mix phoenix.server, but it fails to call in a production environment which is built with distillery.

It makes sense because mix isn't included in the release build. Here is an equivalent

<!--more-->

In `config/config.exs` add the Mix.env
```
config :my_app, :environment, Mix.env()
```

Now replace
```
if Mix.env == :dev do
  # xxxxxx
end
```

with
```
if Application.get_env(:my_app, :environment) == :dev do
  # xxxxxx
end
```