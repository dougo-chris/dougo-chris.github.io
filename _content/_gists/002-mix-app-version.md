---
title: App Version from a git tag and SHA
has_more: true
tags:
  - Elixir
  - Release
---


Convert a git tag and sha into a mix application version

<!--more-->

```
def project do
  [app: :my_app,
   version: app_version(),

def app_version do
  [version, _patch, commit_id] =
    case System.cmd("git", ["describe", "--tags", "--long"]) do
      {tag, 0} ->
        tag
        |> String.trim
        |> String.split("-")

      _ ->
        ["0.0.0", "", "UNKNOWN"]
    end

  "#{version}+ref-#{commit_id}"
end
```