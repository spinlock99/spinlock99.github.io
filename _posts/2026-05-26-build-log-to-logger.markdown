---
layout: post
title:  "Routing Bootleg Build Output Through Elixir Logger"
date:   2026-05-26 14:05:54 -0700
categories: debian
---
{% include google_analytics.html %}

# Routing Bootleg Build Output Through Elixir Logger

The remote build tasks in Bootleg run three noisy commands on the build server:
`mix deps.get`, `mix do clean, compile --force`, and `mix release`. An earlier
commit quieted them by redirecting their output to a temp file on the remote
server:

```bash
MIX_ENV=prod mix deps.get --only=prod >> /tmp/bootleg-myapp-build.log 2>&1
```

That works, but the log ends up stranded on the build server. If a build fails
you have to SSH in and hunt for it. A better approach is to route the output
through Elixir's built-in Logger — suppressed by default, but configurable to
write to any backend.

## Remove the Shell Redirections

The first step is to strip the `>> ... 2>&1` redirections from the build
commands in `lib/bootleg/tasks/build/remote.exs`, and remove the `build_log`
variable and the info message that printed its path:

```elixir
# Before
app_name = Config.app()
build_log = "/tmp/bootleg-#{app_name}-build.log"

UI.info("⚡ Building on remote server with mix env #{mix_env}...")
UI.info("   Build log: #{build_log}")

remote :build, cd: source_path do
  "MIX_ENV=#{mix_env} mix deps.get --only=#{mix_env} >> #{build_log} 2>&1"
  "MIX_ENV=#{mix_env} mix do clean, compile --force >> #{build_log} 2>&1"
end

# After
UI.info("⚡ Building on remote server with mix env #{mix_env}...")

remote :build, cd: source_path do
  "MIX_ENV=#{mix_env} mix deps.get --only=#{mix_env}"
  "MIX_ENV=#{mix_env} mix do clean, compile --force"
end
```

Same change for `mix release` in the `:remote_generate_release` task.

## Route SSH Output Through Logger

With the redirections gone, SSH now receives the build output back on the local
side. The `capture/3` function in `lib/bootleg/ssh.ex` handles incoming SSH
data. Previously it passed stdout to `UI.puts_recv/2` (which writes directly to
IO) and silently dropped stderr. Now both go through `Logger.debug`:

```elixir
require Logger

# stdout (device 0)
{:data, _, 0, data} ->
  {buffer, partial_buffer, data} =
    buffer_complete_lines(data, :stdout, buffer, partial_buffer)

  Logger.debug(fn -> "[#{host.name}] #{String.trim_trailing(data)}" end)
  {buffer, status, partial_buffer}

# stderr (device 1)
{:data, _, 1, data} ->
  {buffer, partial_buffer, data} =
    buffer_complete_lines(data, :stderr, buffer, partial_buffer)

  Logger.debug(fn -> "[#{host.name}][stderr] #{String.trim_trailing(data)}" end)
  {buffer, status, partial_buffer}
```

The lazy-function form (`fn -> ... end`) avoids building the string when the
Logger level is above `:debug`, which is the common case.

## Behaviour

With Logger at its default `:info` level, build output is suppressed — same
as before when it disappeared into a temp file. To see it, configure Logger:

```elixir
config :logger, level: :debug
```

Build output then appears as `[debug]` lines, and any Logger backend
(file, remote aggregator, etc.) captures it automatically. Stderr, which was
previously silently dropped, now surfaces at the same level.

## Results

```
158 tests, 0 failures, 17 skipped
```

No temp files on the build server, no stranded logs, and build output is now a
first-class citizen of the application's logging infrastructure.
