---
layout: post
title:  "Build and Deploy"
date:   2025-09-23 07:00:00 -0700
categories: debian
---
{% include google_analytics.html %}

# Build

I've changed directions since last time. Unlike Distilery, Mix Releases aren't
already tarred and gzipped. I had just been copying the entire release directory
but this caused problems in the Deploy phase. Instead of rewriting all of the
deploy code, we can simplify the port of Bootleg to use Mix Releases by creating
a tar archive that can take it's place in the Deploy process.

> ../bootleg/lib/bootleg/tasks/deploy.exs

Swat out the Distillery Release...

```
remote :build, cd: source_path do
    "MIX_ENV=#{mix_env} mix distillery.release #{release_args}"
```

With a Mix Release.

```
remote :build, cd: source_path do
    "MIX_ENV=#{mix_env} mix release #{release_args}"
end
```

However, `mix release` does not create a `.tar.gz` archive as Distillery did.
So, we will create it ourselves to give `bootstrap` the archive is expects...

```
source_path =
    Path.join([
      config({:ex_path, ""}),
      "/_build/#{mix_env}/rel/"
    ])

  remote :build, cd: source_path  do
    "tar -czvf #{app_name}.tar.gz #{app_name}/"
  end
```

Now, we just have to modify `download_release` to stop looking in the old location...

```
task :download_release do
    remote_path =
        Path.join(
                source_path,
                "_build/#{mix_env}/rel/#{app_name}/releases/#{app_version}/#{app_name}.tar.gz"
                )
```

And, look for it in it's new location.

```
task :download_release do
    remote_path =
        Path.join(
                source_path,
                "_build/#{mix_env}/rel/#{app_name}.tar.gz"
                )
```

Now, we are ready to configure Bootleg to Build the release on the remote host.

> config/deploy.exs

```
role :build, "localhost", workspace: "/home/builder/build",
                          user: "builder",
                          identity: "~/.ssh/id_ed25519",
                          silently_accept_hosts: true

task :build_phoenix do
    mix_env = config({:mix_env, "prod"})
    source_path = config({:ex_path, ""})

    UI.info("Building Phoenix..")

    remote :build, cd: source_path do
        "MIX_ENV=#{mix_env} mix assets.deploy"
        "MIX_ENV=#{mix_env} mix phx.gen.release"
    end
end

before_task(:remote_generate_release, :build_phoenix)
```

Notice that the user `builder` is being used to build the release in a folder
in its home directory. We are also using `ed25519` to encrypt our ssh
credentials which requires `axelson`'s `support-erlang-otp-25` branch.

Also, we have defined the `build_phoenix` task which runs the Phoenix specific
Mix tasks to generate assets and helper functions for Phoenix apps.

> mix bootleg.build

There are lots of warnings and debug statements which make the output a bit
harder to follow. I'll be silencing those in an upcoming PR. But, for now, you
can run the build which will ssh into `builder@localhost`, run the build in
`/home/builder` then copy the release archvie to `releases/`.

```
spinlock@derico:~/src/bootleg_test$ mix bootleg.build
==> bootleg
     warning: expected a module (an atom) when invoking file/0 in expression:

         __CALLER__.file()

     hint: "var.field" (without parentheses) means "var" is a map() while "var.fun()" (with parentheses) means "var" is an atom()

     typing violation found at:
     │
 265 │     file = __CALLER__.file()
     │                       ~
     │
     └─ (bootleg 0.13.0) lib/bootleg/dsl.ex:265:23: Bootleg.DSL.task/3

     warning: Tuple.append/2 is deprecated. Use insert_at instead
     │
 104 │       |> Tuple.append(host)
     │                ~
     │
     └─ (bootleg 0.13.0) lib/bootleg/ssh.ex:104:16: Bootleg.SSH.run/2

     warning: redefining @doc attribute previously set at line 116.

     Please remove the duplicate docs. If instead you want to override a previously defined @doc, attach the @doc attribute to a function head (the function signature not followed by any do-block). For example:

         @doc """
         new docs
         """
         def puts_send(...)

     │
 125 │   @doc """
     │   ~~~~~~~~
     │
     └─ (bootleg 0.13.0) lib/ui.ex:125: Bootleg.UI.puts_send/2

     warning: redefining @doc attribute previously set at line 136.

     Please remove the duplicate docs. If instead you want to override a previously defined @doc, attach the @doc attribute to a function head (the function signature not followed by any do-block). For example:

         @doc """
         new docs
         """
         def puts_recv(...)

     │
 143 │   @doc """
     │   ~~~~~~~~
     │
     └─ (bootleg 0.13.0) lib/ui.ex:143: Bootleg.UI.puts_recv/1

     warning: redefining @doc attribute previously set at line 153.

     Please remove the duplicate docs. If instead you want to override a previously defined @doc, attach the @doc attribute to a function head (the function signature not followed by any do-block). For example:

         @doc """
         new docs
         """
         def puts_recv(...)

     │
 161 │   @doc """
     │   ~~~~~~~~
     │
     └─ (bootleg 0.13.0) lib/ui.ex:161: Bootleg.UI.puts_recv/2

warning: using module.function() notation (with parentheses) to fetch map field :file is deprecated, you must remove the parentheses: map.field
  (bootleg 0.13.0) lib/bootleg/dsl.ex:265: Bootleg.DSL."MACRO-task"/4
  (elixir 1.18.4) src/elixir_dispatch.erl:249: :elixir_dispatch.expand_macro_fun/7
  (elixir 1.18.4) src/elixir_dispatch.erl:118: :elixir_dispatch.dispatch_import/6
  (elixir 1.18.4) src/elixir_expand.erl:625: :elixir_expand.expand_block/5
  (elixir 1.18.4) src/elixir_expand.erl:48: :elixir_expand.expand/3

warning: using module.function() notation (with parentheses) to fetch map field :file is deprecated, you must remove the parentheses: map.field
  (bootleg 0.13.0) lib/bootleg/dsl.ex:135: Bootleg.DSL.add_callback/4
  (elixir 1.18.4) src/elixir_dispatch.erl:249: :elixir_dispatch.expand_macro_fun/7
  (elixir 1.18.4) src/elixir_dispatch.erl:118: :elixir_dispatch.dispatch_import/6
  (elixir 1.18.4) src/elixir_expand.erl:634: :elixir_expand.expand_block/5
  (elixir 1.18.4) src/elixir_expand.erl:48: :elixir_expand.expand/3

Starting remote build for production environment
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /home/builder/build)

09:53:57.753 [debug] client will use strict KEX ordering
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env git init)

09:53:57.906 [debug] client will use strict KEX ordering
[localhost ] Initialized empty Git repository in /home/builder/build/.git/
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env git config receive.denyCurrentBranch ignore)

09:53:58.047 [debug] client will use strict KEX ordering
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /home/builder/build)

09:53:58.223 [debug] client will use strict KEX ordering
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env rm -rvf *)

09:53:58.388 [debug] client will use strict KEX ordering
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

Pushing new commits with git to: builder@localhost
Warning: Permanently added 'localhost' (ED25519) to the list of known hosts.
To ssh://localhost/home/builder/build
 * [new branch]      master -> master

Resetting remote hosts to refspec "master"
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /home/builder/build)

09:53:58.835 [debug] client will use strict KEX ordering
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env git reset --hard master)

09:53:59.001 [debug] client will use strict KEX ordering
[localhost ] HEAD is now at c565f05 Configure Deploy
Building on remote server with mix env prod...
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /home/builder/build)

09:53:59.179 [debug] client will use strict KEX ordering
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env MIX_ENV=prod mix local.rebar --force)

09:53:59.329 [debug] client will use strict KEX ordering
[localhost ] * creating /home/builder/.asdf/installs/elixir/1.18.4-otp-28/.mix/elixir/1-18-otp-28/rebar3
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env MIX_ENV=prod mix local.hex --if-missing --force)

09:54:01.622 [debug] client will use strict KEX ordering
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env MIX_ENV=prod mix deps.get --only=prod)

09:54:02.235 [debug] client will use strict KEX ordering
[localhost ] * Getting heroicons (https://github.com/tailwindlabs/heroicons.git - v2.2.0)
[localhost ] remote: Enumerating objects: 9314, done.
remote: Counting objects: 100% (9314/9314), done. 4)
<--snip-->
[localhost ] * Getting ecto (Hex package)
[localhost ] * Getting phoenix_pubsub (Hex package)
[localhost ] * Getting websock_adapter (Hex package)
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env MIX_ENV=prod mix do clean, compile --force)

09:54:05.017 [debug] client will use strict KEX ordering
[localhost ] ==> mime
[localhost ] Compiling 1 file (.ex)
[localhost ] Generated mime app
<--snip-->
[localhost ] ==> bootleg_test
[localhost ] Compiling 15 files (.ex)
[localhost ] Generated bootleg_test app
Running Phoenix Tasks..
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /home/builder/build)

09:54:26.281 [debug] client will use strict KEX ordering
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env MIX_ENV=prod mix assets.deploy)

09:54:26.445 [debug] client will use strict KEX ordering
[localhost ]
[localhost ] 09:54:27.185 [debug] Downloading tailwind from https://github.com/tailwindlabs/tailwindcss/releases/download/v4.1.7/tailwindcss-linux-x64
[localhost ] ≈ tailwindcss v4.1.7
[localhost ] /*! 🌼 daisyUI 5.0.35 */
[localhost ] Done in 101ms
[localhost ]
[localhost ] 09:54:45.176 [debug] Downloading esbuild from https://registry.npmjs.org/@esbuild/linux-x64/0.25.4
[localhost ]
[localhost ]   ../priv/static/assets/js/app.js  129.4kb
[localhost ]
[localhost ] ⚡ Done in 29ms
[localhost ] Check your digested files at "priv/static"
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env MIX_ENV=prod mix phx.gen.release)

09:54:46.201 [debug] client will use strict KEX ordering
[localhost ] * creating rel/overlays/bin/server
[localhost ] * creating rel/overlays/bin/server.bat
[localhost ] * creating rel/overlays/bin/migrate
[localhost ] * creating rel/overlays/bin/migrate.bat
[localhost ] * creating lib/bootleg_test/release.ex
[localhost ]
[localhost ] Your application is ready to be deployed in a release!
[localhost ]
[localhost ] See https://hexdocs.pm/mix/Mix.Tasks.Release.html for more information about Elixir releases.
[localhost ]
[localhost ] Here are some useful release commands you can run in any release environment:
[localhost ]
[localhost ]     # To build a release
[localhost ]     mix release
[localhost ]
[localhost ]     # To start your system with the Phoenix server running
[localhost ]     _build/dev/rel/bootleg_test/bin/server
[localhost ]
[localhost ]     # To run migrations
[localhost ]     _build/dev/rel/bootleg_test/bin/migrate
[localhost ]
[localhost ] Once the release is running you can connect to it remotely:
[localhost ]
[localhost ]     _build/dev/rel/bootleg_test/bin/bootleg_test remote
[localhost ]
[localhost ] To list all commands:
[localhost ]
[localhost ]     _build/dev/rel/bootleg_test/bin/bootleg_test
Generating release...
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /home/builder/build)

09:54:47.113 [debug] client will use strict KEX ordering
[localhost ] cd /home/builder/build && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env MIX_ENV=prod mix release --quiet)

09:54:47.263 [debug] client will use strict KEX ordering
[localhost ] Compiling 1 file (.ex)
[localhost ] Generated bootleg_test app
[localhost ] * using config/runtime.exs to configure the release at runtime
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /home/builder/build)

09:54:48.583 [debug] client will use strict KEX ordering
[localhost ] cd /home/builder/build/_build/prod/rel && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env tar -czvf bootleg_test.tar.gz bootleg_test/)

09:54:48.741 [debug] client will use strict KEX ordering
[localhost ] bootleg_test/
[localhost ] bootleg_test/bin/
[localhost ] bootleg_test/bin/server.bat
[localhost ] bootleg_test/bin/migrate.bat
[localhost ] bootleg_test/bin/server
[localhost ] bootleg_test/bin/bootleg_test.bat
[localhost ] bootleg_test/bin/bootleg_test
[localhost ] bootleg_test/bin/migrate
[localhost ] bootleg_test/releases/
[localhost ] bootleg_test/releases/COOKIE
[localhost ] bootleg_test/releases/start_erl.data
[localhost ] bootleg_test/releases/0.1.0/
<--snip-->
[localhost ] bootleg_test/lib/db_connection-2.8.1/ebin/Elixir.DBConnection.EncodeError.beam
Downloading release archive
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /home/builder/build)

09:54:52.107 [debug] client will use strict KEX ordering
[localhost ] DOWNLOAD /home/builder/build/_build/prod/rel/bootleg_test.tar.gz -> releases/0.1.0.tar.gz

09:54:52.276 [debug] client will use strict KEX ordering
Saved: releases/0.1.0.tar.gz
```

# Deploy

> config/deploy/production.exs

Again, we are using the `builder` user to `ssh` into `localhost` to deploy the
app into `/var/www/bootleg` which is where SystemD will expect to find it. This
directory must be created by `root` and then `chown builder:builder` so that it
can be used for deployment.

```
role :app, ["localhost"],
  workspace: "/var/www/bootleg",
  user: "builder",
  identity: "~/.ssh/id_ed25519",
  silently_accept_hosts: true
```

> ../bootleg/lib/bootleg/tasks/deploy.exs

Nothing to do here! Because we recreated the expected archive, Bootleg didn't
need any modifications to the deploy task.

> mix bootleg.deploy

```
spinlock@derico:~/src/bootleg_test$ mix bootleg.deploy
==> bootleg
     warning: expected a module (an atom) when invoking file/0 in expression:

         __CALLER__.file()

     hint: "var.field" (without parentheses) means "var" is a map() while "var.fun()" (with parentheses) means "var" is an atom()

     typing violation found at:
     │
 265 │     file = __CALLER__.file()
     │                       ~
     │
     └─ (bootleg 0.13.0) lib/bootleg/dsl.ex:265:23: Bootleg.DSL.task/3

     warning: Tuple.append/2 is deprecated. Use insert_at instead
     │
 104 │       |> Tuple.append(host)
     │                ~
     │
     └─ (bootleg 0.13.0) lib/bootleg/ssh.ex:104:16: Bootleg.SSH.run/2

     warning: redefining @doc attribute previously set at line 116.

     Please remove the duplicate docs. If instead you want to override a previously defined @doc, attach the @doc attribute to a function head (the function signature not followed by any do-block). For example:

         @doc """
         new docs
         """
         def puts_send(...)

     │
 125 │   @doc """
     │   ~~~~~~~~
     │
     └─ (bootleg 0.13.0) lib/ui.ex:125: Bootleg.UI.puts_send/2

     warning: redefining @doc attribute previously set at line 136.

     Please remove the duplicate docs. If instead you want to override a previously defined @doc, attach the @doc attribute to a function head (the function signature not followed by any do-block). For example:

         @doc """
         new docs
         """
         def puts_recv(...)

     │
 143 │   @doc """
     │   ~~~~~~~~
     │
     └─ (bootleg 0.13.0) lib/ui.ex:143: Bootleg.UI.puts_recv/1

     warning: redefining @doc attribute previously set at line 153.

     Please remove the duplicate docs. If instead you want to override a previously defined @doc, attach the @doc attribute to a function head (the function signature not followed by any do-block). For example:

         @doc """
         new docs
         """
         def puts_recv(...)

     │
 161 │   @doc """
     │   ~~~~~~~~
     │
     └─ (bootleg 0.13.0) lib/ui.ex:161: Bootleg.UI.puts_recv/2

warning: using module.function() notation (with parentheses) to fetch map field :file is deprecated, you must remove the parentheses: map.field
  (bootleg 0.13.0) lib/bootleg/dsl.ex:265: Bootleg.DSL."MACRO-task"/4
  (elixir 1.18.4) src/elixir_dispatch.erl:249: :elixir_dispatch.expand_macro_fun/7
  (elixir 1.18.4) src/elixir_dispatch.erl:118: :elixir_dispatch.dispatch_import/6
  (elixir 1.18.4) src/elixir_expand.erl:625: :elixir_expand.expand_block/5
  (elixir 1.18.4) src/elixir_expand.erl:48: :elixir_expand.expand/3

warning: using module.function() notation (with parentheses) to fetch map field :file is deprecated, you must remove the parentheses: map.field
  (bootleg 0.13.0) lib/bootleg/dsl.ex:135: Bootleg.DSL.add_callback/4
  (elixir 1.18.4) src/elixir_dispatch.erl:249: :elixir_dispatch.expand_macro_fun/7
  (elixir 1.18.4) src/elixir_dispatch.erl:118: :elixir_dispatch.dispatch_import/6
  (elixir 1.18.4) src/elixir_expand.erl:634: :elixir_expand.expand_block/5
  (elixir 1.18.4) src/elixir_expand.erl:48: :elixir_expand.expand/3

Uploading release archive
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /var/www/bootleg)

10:10:38.933 [debug] client will use strict KEX ordering
[localhost ] UPLOAD releases/0.1.0.tar.gz -> /var/www/bootleg/bootleg_test.tar.gz

10:10:39.257 [debug] client will use strict KEX ordering
Unpacking release archive
warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:46: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

warning: IO.binread(device, :all) is deprecated, use IO.binread(device, :eof) instead
  (elixir 1.18.4) lib/io.ex:218: IO.binread/2
  (ssh_client_key_api 0.2.1) lib/ssh_client_key_api.ex:47: SSHClientKeyAPI.with_options/1
  (bootleg 0.13.0) lib/bootleg/ssh.ex:300: Bootleg.SSH.ssh_transform_opt/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (elixir 1.18.4) lib/enum.ex:1714: Enum."-map/2-lists^map/1-1-"/2
  (bootleg 0.13.0) lib/bootleg/ssh.ex:248: Bootleg.SSH.ssh_opts/1

[localhost ] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /var/www/bootleg)

10:10:39.515 [debug] client will use strict KEX ordering
[localhost ] cd /var/www/bootleg && (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env tar -zxvf bootleg_test.tar.gz)

10:10:39.662 [debug] client will use strict KEX ordering
[localhost ] bootleg_test/
[localhost ] bootleg_test/bin/
[localhost ] bootleg_test/bin/server.bat
[localhost ] bootleg_test/bin/migrate.bat
[localhost ] bootleg_test/bin/server
[localhost ] bootleg_test/bin/bootleg_test.bat
[localhost ] bootleg_test/bin/bootleg_test
[localhost ] bootleg_test/bin/migrate
<--snip-->
[localhost ] bootleg_test/lib/db_connection-2.8.1/ebin/Elixir.DBConnection.TelemetryListener.beam
[localhost ] bootleg_test/lib/db_connection-2.8.1/ebin/Elixir.DBConnection.EncodeError.beam
Unpacked release archive
```

# Next Steps

* Get it running with SystemD.
* Put it behind Nginx.
* Self Signed Cert.
