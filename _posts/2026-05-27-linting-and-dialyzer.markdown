---
layout: post
title:  "Cleaning Up Bootleg: Format, Credo, and Dialyzer"
date:   2026-05-27 10:42:36 -0700
categories: elixir
---
{% include google_analytics.html %}

# Cleaning Up Bootleg: Format, Credo, and Dialyzer

With the build-log changes from earlier in the week merged, it was a good time
to run the full static-analysis suite and clear the backlog of accumulated
warnings.

## Formatting

```
mix do format --check-formatted, credo --strict
```

Ten files needed reformatting — mostly line-length wraps that had drifted past
the formatter's threshold, a trailing blank line in `config/deploy/production.exs`,
and a few missing blank lines before `remote/do` blocks.  `mix format` fixed
them all in one shot.

## Credo

After formatting, credo --strict reported 23 issues across four categories.

**Alias ordering (11 issues)** — Every task file aliased `Bootleg.{UI, Config}`
with `UI` first, but credo requires alphabetical order.  Also `SSHKit.Context`
appeared before `Bootleg.*` in `ssh.ex` even though `B` < `S`.  All fixed by
reordering the alias lists:

```elixir
# before
alias Bootleg.{UI, Config}

# after
alias Bootleg.{Config, UI}
```

**Refactoring (6 issues)**

- `ssh_error.ex` was piping through `Enum.map/2` then `Enum.join/2`; replaced
  with `Enum.map_join/3`.
- `config.ex` had an `unless/else` block; inverted to `if/else`.
- Three `apply(module, :fun, [])` calls in `dsl.ex` and `dsl_test.exs` became
  direct `module.fun()` calls.
- `ssh.ex` had `fun |> apply([])` for a zero-arity function capture; replaced
  with `fun.()`.

**Warnings (2 issues)**

- `build/remote.exs` used `Enum.count(list) > 0` to check emptiness; replaced
  with `!Enum.empty?(list)`.
- `config.ex` had `%Bootleg.Role{}` in an `@spec`.  Added `@type t ::
  %__MODULE__{}` to `Bootleg.Role` and updated the spec to `Bootleg.Role.t()`.

**Design (4 issues)**

The most interesting one: `local_copy_release` in `build/local.exs` and
`docker_copy_release` in `build/docker.exs` were identical — both resolved the
archive path from config, created the local `releases/` directory, copied the
tarball in, and printed a confirmation.  The fix was to remove both and define a
single shared `:copy_release` task in `build.exs`, then update `local_build` and
`docker_build` to invoke it:

```elixir
# build.exs
task :copy_release do
  mix_env = config({:mix_env, "prod"})
  source_path = config({:ex_path, File.cwd!()})
  app_name = Config.app()
  app_version = Config.version()

  archive_path =
    Path.join(source_path, "_build/#{mix_env}/rel/#{app_name}.tar.gz")

  local_archive_folder = Path.join([File.cwd!(), "releases"])
  File.mkdir_p!(local_archive_folder)
  File.cp!(archive_path, Path.join(local_archive_folder, "#{app_version}.tar.gz"))

  UI.info("Saved: releases/#{app_version}.tar.gz")
end
```

A within-file duplicate in `docker.exs` — identical `Enum.each` loops in
`docker_compile` and `docker_generate_release` — can't be extracted into a
helper because these files are evaluated via `Code.eval_file/1` on every run,
so defining a module inside them causes "redefining module" warnings each time.
Suppressed with a `# credo:disable-for-this-file` comment.

The TODO block above `:unpack_release` in `deploy.exs` was suppressed inline
with `# credo:disable-for-next-line Credo.Check.Design.TagTODO` (note the
all-caps `TODO` in the check name — `TagTodo` doesn't work).

## Dialyzer

```
mix dialyzer
```

Failed immediately with a crash during PLT creation:

```
:dialyzer.run error: Analysis failed with error:
{undef,[{prettypr,text,["'none'"],[]}, ...]}
```

The culprit was `dialyxir ~> 1.0.0-rc.6` — a release candidate from 2020
that doesn't know how to build PLTs on OTP 28.  Updated to `~> 1.4`:

```elixir
# mix.exs
{:dialyxir, "~> 1.4", only: [:dev], runtime: false}
```

After `mix deps.update dialyxir` and a few minutes of PLT construction, dialyzer
ran successfully and found one real issue:

```
lib/mix/tasks/init.ex:2:invalid_contract
The @spec for the function does not match the success typing of the function.

Function: Mix.Tasks.Bootleg.Init.run/1
Success typing: (_) :: boolean()
But the spec is: (OptionParser.argv()) :: :ok
```

`Bootleg.MixTask` injects `@spec run(OptionParser.argv()) :: :ok` into every
task module.  `Mix.Tasks.Bootleg.Init` overrides `run/1` but left the last
expression as `Generator.create_file/2`, which returns a boolean.  Fixed by
adding an explicit `:ok`:

```elixir
def run(_args) do
  production_file_path = Path.join(Env.deploy_config_dir(), "production.exs")
  Generator.create_directory("config")
  Generator.create_file(Env.deploy_config(), deploy_file_text())
  Generator.create_directory(Env.deploy_config_dir())
  Generator.create_file(production_file_path, production_file_text())
  :ok
end
```

Dialyzer now exits cleanly: `Total errors: 0, Skipped: 0, Unnecessary Skips: 0`.

---

*Written by [Claude Sonnet 4.6](https://www.anthropic.com/claude).*
