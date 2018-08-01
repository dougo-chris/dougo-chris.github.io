---
title:  MySQL Backup to S3
tags:
  - Elixir
  - MySQL
  - Script
---

Here's a script I wrote to backup my MySQL database and upload to S3. It's. based on a Ruby version I've been using for years.

<!--more-->

I'm using Ecto and ExAws libraries as well as the following System ENV variables

```
BACKUP_AWS_BUCKET
BACKUP_DIRECTORY
BACKUP_AWS_ACCESS_KEY_ID
BACKUP_AWS_SECRET_ACCESS_KEY
```

The tasks are

```
myapp.mysql.backup    <- backup the database and upload to S3
myapp.mysql.download  <- download the latest version from S3
myapp.mysql.restore.  <- restore the latest version
```

```
defmodule Mix.Tasks.MyApp.Mysql do
  @moduledoc false

  alias MyApp.Repo
  alias ExAws.S3
  alias ExAws.S3.Upload

  defmodule Backup do
    @moduledoc false
    @shortdoc "MySql Backup"

    use Mix.Task
    alias Mix.Tasks.MyApp.Mysql

    def run(_args) do
      Mysql.s3_start()

      path = Mysql.path()
      filename = Mysql.filename()

      IO.puts("BACKUP : " <> filename)

      opts =
        (Mysql.mysql_opts() ++
           [
             "--default-character-set=utf8",
             "--skip-lock-tables",
             "--single-transaction",
             Mysql.database()
           ])
        |> Enum.join(" ")

      _ =
        "mysqldump #{opts} | bzip2 > #{path}/#{filename}"
        |> String.to_charlist()
        |> :os.cmd()

      # if Application.get_env(:linklab, :environment) == :production do
      if Mix.env() == :production do
        Mysql.s3_upload(path, filename)
        File.rm!(path <> "/" <> filename)
      end

      IO.puts("BACKUP : SUCCESS #{filename}")

      :ok
    end
  end

  defmodule Download do
    @moduledoc false
    @shortdoc "MySql Download"

    use Mix.Task
    alias Mix.Tasks.MyApp.Mysql
    alias ExAws.S3

    def run(_args) do
      Mysql.s3_start()

      body =
        "BACKUP_AWS_BUCKET"
        |> System.get_env()
        |> S3.list_objects(delimiter: "/", prefix: "production")
        |> ExAws.request!(Mysql.s3_secrets())

      %{body: %{contents: contents}} = body

      %{key: filename} =
        contents
        |> Enum.sort_by(fn content -> content[:key] end)
        |> Enum.reverse()
        |> (&hd(&1)).()

      path = Mysql.path()

      if File.exists?(path <> "/" <> filename) do
        File.rm!(path <> "/" <> filename)
      end

      IO.puts("DOWNLOAD : " <> path <> "/" <> filename)

      "BACKUP_AWS_BUCKET"
      |> System.get_env()
      |> S3.download_file(filename, path <> "/" <> filename)
      |> ExAws.request!(Mysql.s3_secrets())

      IO.puts("DOWNLOAD : SUCCESS")
      :ok
    end
  end

  defmodule Restore do
    @moduledoc false
    @shortdoc "MySql Restore"

    use Mix.Task
    alias Mix.Tasks.MyApp.Mysql

    def run(args) do

      env = case args do
          [] -> "prod"
          [env | _] -> env
        end

      path = Mysql.path()
      filenames =
        "#{path}/#{env}-*.sql.bz2"
        |> Path.wildcard()
        |> Enum.sort()
        |> Enum.reverse()

      case filenames do
        [] ->
          IO.puts(
            "NO FILES TO RESTORE : " <>
            path <>
            "/#{env}-*.sql.bz2"
          )

        [filename | _] ->
          IO.puts("RESTORE : #{filename}")

          opts =
            Mysql.mysql_opts()
            |> Enum.join(" ")

          "bzip2 -dc #{filename} | mysql #{opts} --database=#{Mysql.database()}"
          |> String.to_charlist()
          |> :os.cmd()

          IO.puts("RESTORE : SUCCESS")
      end

      :ok
    end
  end

  def path do
    path = System.get_env("BACKUP_DIRECTORY")

    unless File.exists?(path) do
      File.mkdir_p!(path)
    end

    path
  end

  def filename do
    {:ok, dt} =
      :seconds
      |> :os.system_time()
      |> DateTime.from_unix()

    [
      Mix.env(),
      dt.year |> Integer.to_string() |> String.pad_leading(4, "0"),
      dt.month |> Integer.to_string() |> String.pad_leading(2, "0"),
      dt.day |> Integer.to_string() |> String.pad_leading(2, "0"),
      dt.hour |> Integer.to_string() |> String.pad_leading(2, "0"),
      dt.minute |> Integer.to_string() |> String.pad_leading(2, "0"),
      dt.second |> Integer.to_string() |> String.pad_leading(2, "0")
    ]
    |> Enum.join("-")
    |> (&(&1 <> ".sql.bz2")).()
  end

  def s3_start do
    {:ok, _started} = Application.ensure_all_started(:hackney)
  end

  def s3_secrets do
    [
      access_key_id: System.get_env("BACKUP_AWS_ACCESS_KEY_ID"),
      secret_access_key: System.get_env("BACKUP_AWS_SECRET_ACCESS_KEY")
    ]
  end

  def s3_upload(path, filename) do
    (path <> "/" <> filename)
    |> Upload.stream_file()
    |> S3.upload(System.get_env("BACKUP_AWS_BUCKET"), filename)
    |> ExAws.request!(s3_secrets())
  end

  def mysql_opts do
    Repo.config()
    |> Enum.map(fn
      {_, nil} ->
        nil

      {_, ""} ->
        nil

      {key, value} ->
        case key do
          :username -> "--user=#{value}"
          :password -> "--password=#{value}"
          :hostname -> "--host=#{value}"
          :port -> "--port=#{value}"
          _ -> nil
        end
    end)
    |> Enum.reject(&is_nil/1)
  end

  def database do
    Repo.config()[:database]
  end
end
```