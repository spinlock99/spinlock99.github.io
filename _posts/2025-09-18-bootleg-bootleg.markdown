---
layout: post
title:  "Bootleg Bootleg"
date:   2025-09-18 07:00:00 -0700
categories: debian
---
{% include google_analytics.html %}

# Bootleg

We need to add a custom installation of `bootleg` because it's no longer maintined.

> mix.ex

```
 defp deps do
 [
   {:phoenix, "~> 1.8.1"},
   <snip>
   {:bootleg, path: "../bootleg", only: :dev}
 ]
end
```

We also need to add a custom version of `ssh_client_key_api to use ed_25591 keys
in ssh.

> bootleg/mix.ex

```
  defp deps do
   [
     {:sshkit, "0.3.0"},
     {:ssh_client_key_api, github: "axelson/ssh_client_key_api", branch: "support-erlang-otp-25"},
     {:credo, "~> 1.7", only: [:dev, :test], runtime: false},
     <snip>
   ]
 end
 ```

 We've also updated credo to the latest version.

## Update Remote Generate Release Task

Use Mix Releases rather than Distillery Releaseses:

```
 spinlock@derico:~/src/bootleg$ git diff master
diff --git a/lib/bootleg/tasks/build/remote.exs b/lib/bootleg/tasks/build/remote.exs
index 4723da7..e2854e8 100644
--- a/lib/bootleg/tasks/build/remote.exs
+++ b/lib/bootleg/tasks/build/remote.exs
@@ -69,7 +69,7 @@ task :remote_generate_release do
   UI.info("Generating release...")

   remote :build, cd: source_path do
-    "MIX_ENV=#{mix_env} mix distillery.release #{release_args}"
+    "MIX_ENV=#{mix_env} mix release #{release_args}"
   end
 end

@@ -120,18 +120,18 @@ task :download_release do
   remote_path =
     Path.join(
       source_path,
-      "_build/#{mix_env}/rel/#{app_name}/releases/#{app_version}/#{app_name}.tar.gz"
+      "_build/#{mix_env}/rel/#{app_name}"
     )

   local_archive_folder = "#{File.cwd!()}/releases"
-  local_path = Path.join(local_archive_folder, "#{app_version}.tar.gz")
+  local_path = Path.join(local_archive_folder, "#{app_version}")

   UI.info("Downloading release archive")
   File.mkdir_p!(local_archive_folder)

   download(:build, remote_path, local_path)

-  UI.info("Saved: releases/#{app_version}.tar.gz")
+  UI.info("Saved: releases/#{app_version}")
 end
```

## Git Ignore Releases

Add releases directory to the git ignore file:

> .gitignore

```
# Releases
/releases/
```

## Build Role

Bootleg will compile the release with the `build` role. It will `ssh` into
the build server as the specified `user`. It then compiles the release in the
`workspace` directory. We have also specified `ed25519` encryption for `ssh`.

> config/deploy.exs

```
use Bootleg.DSL

# Configure the following roles to match your environment.
# `build` defines what remote server your distillery release should be built on.
#
# Some available options are:
#  - `user`: ssh username to use for SSH authentication to the role's hosts
#  - `password`: password to be used for SSH authentication
#  - `identity`: local path to an identity file that will be used for SSH authentication instead of a password
#  - `workspace`: remote file system path to be used for building and deploying this Elixir project
use Bootleg.DSL
alias Bootleg.{Config, UI}

role :build, "localhost", workspace: "/home/builder/build",
                          user: "builder",
                          identity: "~/.ssh/id_ed25519",
                          silently_accept_hosts: true

task :run_phoenix_tasks do
  mix_env = config({:mix_env, "prod"})
  source_path = config({:ex_path, ""})

  UI.info("Running Phoenix Tasks..")

  remote :build, cd: source_path do
    "MIX_ENV=#{mix_env} mix assets.deploy"
    "MIX_ENV=#{mix_env} mix phx.gen.release"
  end
end

before_task(:remote_generate_release, :run_phoenix_tasks)

```

> $mix bootleg.build

> $cd releases/0.1.0/bin

> DATABASE_URL=ecto://postgres:postgres@staging.haikuter.com/bootleg_test_prod SECRET_KEY_BASE=<snip> ./server

