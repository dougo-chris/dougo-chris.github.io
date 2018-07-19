---
title:  Elixir Project Structure
tags:
  - Elixir
  - IDE
---

One of the ways I keep up with projects and what things do is to keep code in locations that logically represent the functionality they perform.

From libs, tasks, database schemas, and domain logic all have their place. Here is how I organize my code

<!--more-->

Working with a Phoenix projects the standards all make sense to me

```
/assets   <- web assets
/config   <- configuration between prod, dev and test
/lib      <- My code
/private  <- Files to include in a release
/test     <- Test cases
```

### Mix Tasks
At this level I add another directory for the mix tasks I use in the development of my projects.
```
/tasks
```
I also need to add the directory to my mix.exs file
```
defp elixirc_paths(_), do: ["lib", "task"]
```

### Project files in lib
```
/lib/schema   <- Table definitions
/lib/domain   <- Domain and business logic
/lib/web      <- Endpoints for web interactions
/lib/workers  <- Background workers
```

The important thing here is I separate the database schema from the domain logic. The domain logic is the heavy lifter of my code base.

Table definitions are responsible for validating the code we write is valid. The schema does NOT write to the repo.

The domain logic is where I insert, find, update and delete individual or multiple records.

### Domain logic is what it's all about.

By adopting this structure I have been able to keep the web controllers small, the database schema's simple and build different domain objects that help me solve the complex business problems I'm given as a developer