---
layout: post
title:  "Quieting Bootleg Test Output"
date:   2026-05-22 17:36:21 -0700
categories: debian
---
{% include google_analytics.html %}

# Quieting Bootleg Test Output

The bootleg test suite passes clean, but the output is noisy — hundreds of
`[debug] client will use strict KEX ordering` lines from Erlang's `:ssh`
application, plus deprecation warnings from old fixture config files. Time to
clean that up.

## Suppress SSH Debug Noise

The debug messages come from Erlang's `:ssh` app and are logged via Logger.
The fix is to raise the log level to `:warning` during tests so they never
print.

Bootleg had no `config/test.exs`, so we created one:

> bootleg/config/test.exs

```elixir
import Config

config :logger, level: :warning
```

Then wired it in from `config/config.exs`:

> bootleg/config/config.exs

```elixir
import Config

if config_env() == :test, do: import_config("test.exs")
```

The conditional guard is needed because only `test.exs` exists — an
unconditional `import_config` would blow up in `:dev` or `:prod`.

## Fix Mix.Config Deprecation in the Umbra Fixture

The test suite was printing two deprecation warnings during the `init umbrella
project` functional test:

```
warning: use Mix.Config is deprecated. Use the Config module instead
warning: Mix.Config.import_config/1 is deprecated. Use the Config module instead
```

Both came from the `umbra` test fixture's config file. The fix is
straightforward for the first warning — swap `use Mix.Config` for
`import Config`. The second is trickier.

`Mix.Config.import_config/1` supported glob patterns. `Config.import_config/1`
does not — it tries to read the literal path, `*` and all, and raises
`File.Error` if the file doesn't exist. The fix is to expand the glob manually
with `Path.wildcard/1`:

> bootleg/test/fixtures/umbra/config/config.exs

```elixir
# Before
use Mix.Config
import_config "../apps/*/config/config.exs"

# After
import Config

for config <- Path.wildcard(Path.expand("../apps/*/config/config.exs", __DIR__)) do
  import_config config
end
```

`Path.expand/2` resolves the path relative to the config file's own directory
(`__DIR__`), and `Path.wildcard/1` handles the glob. If no app configs exist
the loop is a no-op, matching the original `Mix.Config` behaviour.

## Fix Range Warning

The functional test setup maps over a range to boot N Docker containers:

```elixir
hosts = Enum.map(1..count, fn _ -> init(boot(conf), passphrase: key_passphrase) end)
```

When `count` is `0` (used by tests that don't need a container), `1..0`
triggers a warning because Elixir now requires an explicit step when
`last < first`:

```
warning: Range.new/2 and first..last default to a step of -1 when last < first.
Use Range.new(first, last, -1) or first..last//-1, or pass 1 if that was your intention
```

The intent is clearly "iterate `count` times", so the step should be `+1`.
With `//1`, an empty range is produced when `count` is `0`:

> bootleg/test/support/functional_case.ex

```elixir
hosts = Enum.map(1..count//1, fn _ -> init(boot(conf), passphrase: key_passphrase) end)
```

## Results

```
158 tests, 0 failures, 17 skipped
```

The output is now clean — no debug spam, no deprecation warnings, no Range
warnings.
