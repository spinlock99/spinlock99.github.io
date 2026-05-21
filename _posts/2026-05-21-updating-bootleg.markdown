---
layout: post
title:  "Updating Bootleg"
date:   2026-05-21 07:00:00 -0700
categories: debian
---
{% include google_analytics.html %}

# Updating Bootleg

Bootleg is still working great, but there are a bunch of warnings coming from
the `distillery` dependency that we can clean up. Since we already switched to
`mix release`, we don't need distillery at all anymore. Time to rip it out.

## Remove Distillery

Distillery showed up in two places: the build tasks and the test fixtures.

### Build Tasks

The `docker` and `local` build tasks were still calling `mix distillery.release`.
We swapped them out for `mix release` and added a `tar` step to create the
archive that the deploy task expects, matching what the `remote` build task
was already doing.

> bootleg/lib/bootleg/tasks/build/docker.exs

```elixir
# Before
commands = [
  ["mix", ["distillery.release"] ++ release_args]
]

# After
commands = [
  ["mix", ["release"] ++ release_args],
  ["bash", ["-c", "cd _build/#{mix_env}/rel && tar -czvf #{app_name}.tar.gz #{app_name}/"]]
]
```

The archive path also changed since distillery used to put it in
`releases/#{version}/#{app}.tar.gz` but `mix release` just builds
the release directory and we tar it ourselves.

```elixir
# Before
"_build/#{mix_env}/rel/#{app_name}/releases/#{app_version}/#{app_name}.tar.gz"

# After
"_build/#{mix_env}/rel/#{app_name}.tar.gz"
```

### Test Fixtures

The test fixtures (`build_me`, `n00b`, `task_consumer`) had distillery as a dep
and `build_me` and `bootstraps` had a `rel/config.exs` for distillery release
configuration. We removed all of it.

> test/fixtures/build_me/mix.exs

```elixir
# Before
defp deps do
  [{:distillery, "~> 2.1.0", runtime: false}]
end

# After
defp deps do
  []
end
```

We also deleted the `rel/config.exs` files from `build_me/` and `bootstraps/`
since `mix release` works with zero config for basic apps.

### Fix use Mix.Config

While we were in the fixtures, we fixed the `use Mix.Config` deprecation warning
that had been showing up in the test output. Three fixture config files were
still using the old syntax.

```elixir
# Before
use Mix.Config

# After
import Config
```

## Point bootleg_test at Local bootleg

Now that we're actively working on bootleg, we want `bootleg_test` to use our
local copy instead of pulling from GitHub.

> bootleg_test/mix.exs

```elixir
# Before
{:bootleg, github: "labzero/bootleg", only: :dev}

# After
{:bootleg, path: "../bootleg", only: :dev}
```

Then run `mix deps.get` to pick up the change.

## Bump the Version

With the distillery cleanup done, bump the version to reflect the update.

> bootleg_test/mix.exs

```elixir
version: "0.1.2",
```

## Run the Tests

After all of that, the bootleg test suite still passes clean:

```
158 tests, 0 failures, 17 skipped
```

The 17 skipped tests are the Docker-based functional tests that require Docker
to be running. Everything else is green.

## Next Steps

* Quiet the noisy build output.
* Fix the remaining warnings in bootleg's own code (`Tuple.append/2`,
  `__CALLER__.file()`, duplicate `@doc` attributes).
