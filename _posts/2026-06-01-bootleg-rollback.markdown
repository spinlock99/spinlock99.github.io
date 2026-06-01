---
layout: post
title:  "Bootleg Rollback"
date:   2026-06-01 12:39:50 -0700
categories: elixir
---
{% include google_analytics.html %}

# Bootleg Rollback

One of the most stressful moments in software development is deploying a bad
release. You push the button, something goes wrong, and now you're scrambling
to restore service. Until now, Bootleg had no answer for this — every deploy
simply overwrote the previous one, leaving you with nothing to fall back to.

This post covers the implementation of rollback support in
[bootleg](https://github.com/labzero/bootleg), addressing
[issue #243](https://github.com/labzero/bootleg/issues/243).

## The Old Way

Previously, `mix bootleg.deploy` would upload a tarball to your app server and
unpack it directly into your workspace:

```
/opt/myapp/
└── myapp/         ← overwritten on every deploy
    ├── bin/myapp
    └── releases/
```

Simple, but with a fatal flaw: the moment a new deploy started, the previous
version was gone. Rollback meant either keeping a local copy of old builds, or
re-building from an earlier commit — neither of which you want to do at 2am.

## The New Way

Inspired by Capistrano's release management, each deploy now unpacks into a
timestamped directory, with a `current` symlink pointing at the active release:

```
/opt/myapp/
├── releases/
│   ├── 20260601120000/    ← previous release
│   └── 20260601153012/    ← current release
└── current -> releases/20260601153012/
```

Your app runs from `current/`, and `current` is just a symlink. Rolling back is
as simple as pointing that symlink somewhere else.

## How It Works

### Deploying

The `:unpack_release` task now does four things:

```bash
mkdir -p releases/
tar -zxf myapp.tar.gz -C releases/
ls -td releases/*/ | head -1 | xargs -I{} ln -sfn {} current
rm myapp.tar.gz
```

Each release gets its own directory. The `current` symlink is updated atomically
— there's no window where the app is pointing at a half-extracted release.

The `:start`, `:stop`, `:restart`, and `:ping` tasks all run through
`current/bin/myapp`, so they always target whatever release is active.

### Rolling Back

```bash
mix bootleg.rollback
```

That's it. Under the hood:

```bash
# Verify there's something to roll back to
test $(ls -1d releases/*/ 2>/dev/null | wc -l) -ge 2

# Point current at the second-most-recent release
ls -1dt releases/*/ | tail -2 | head -1 | xargs -I{} ln -sfn {} current

# Delete the bad release
ls -1dt releases/*/ | tail -1 | xargs -I{} rm -rf {}
```

The symlink swap is fast and atomic. Your app is back on the previous release in
under a second, with no re-upload and no re-build.

`ls -1dt releases/*/` lists release directories sorted by modification time,
newest first. `tail -2 | head -1` picks the second entry — the release just
before the current one. After pointing `current` there, the bad release is
deleted so it can't be rolled back to again.

### Keeping History Tidy

By default, all releases are kept. If you want to cap the number of releases
retained on disk, add to your `config/deploy.exs`:

```elixir
config :keep_releases, 5
```

After each successful deploy, anything beyond the five most recent releases is
pruned automatically.

## Breaking Change

This is a breaking change. If you're upgrading an existing Bootleg deployment,
the old flat `myapp/` directory at the workspace root won't work with the new
`current/bin/myapp` paths. You'll need to run a fresh deploy to establish the
`releases/` structure before your app can be managed with the new version.

## The Payoff

Deployments are now recoverable by default. A bad release is no longer a crisis
— it's a one-command fix. And because releases are just directories with a
symlink in front of them, there's no magic: you can inspect, compare, or restore
any of them manually if you need to.

---

*Written by [Claude Sonnet 4.6](https://www.anthropic.com/claude).*
