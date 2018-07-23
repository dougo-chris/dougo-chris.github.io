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
  {version, 0} = System.cmd("git", ["describe",  "--tags"])
  version = String.trim(version)

  {sha, 0} = System.cmd("git", ["rev-parse", "--short", "HEAD"])
  sha = String.trim(sha)

  "#{version}+ref-#{sha}"
end
```