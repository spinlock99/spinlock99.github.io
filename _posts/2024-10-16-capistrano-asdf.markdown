---
layout: post
title:  "Capistrano and ASDF"
date:   2024-10-13 07:00:00 -0700
categories: elixir phoenix gigalixir
---
{% include google_analytics.html %}

## Build on the Staging Server

Fist experiment is to build the release in the `updated` hook of the `deploy`
command.

# Build Tools Use ASDF

The first issue we hit is that our tools are managed with ASDF. We use the
[Capistrano ASDF Plugin](https://github.com/cabesa-collective/capistrano-asdf/)
to install `asdf`. One gotcha in the installation is to copy the `.tool-versions`
file to `/home/builder/`.

Once our tools are installed on `staging.haikuter.com` we're ready to build
the release.

# Modify Deploy to Build Assets on Staging Server

> config/deploy/staging.rb

```
namespace :deploy do
  task :updated do
    SSHKit.config.command_map.prefix[:mix].unshift("#{fetch(:asdf_wrapper_path)}")
    on roles(:app) do |host|
      within release_path do
        SSHKit.config.default_env[:MIX_ENV] = 'prod'
        execute(:mix, 'deps.get --only prod')
        execute(:mix, 'compile')
        execute(:mix, 'assets.deploy')
      end
    end
  end
end
```

> spinlock@Derico:~/src/haikuter$ cap staging deploy

```
00:00 git:wrapper
      01 mkdir -p /tmp
    ✔ 01 builder@staging.haikuter.com 0.695s
      Uploading /tmp/git-ssh-2cad1b219174965a5c6a.sh 100.0%
      02 chmod 700 /tmp/git-ssh-2cad1b219174965a5c6a.sh
    ✔ 02 builder@staging.haikuter.com 0.047s
00:00 git:check
      01 git ls-remote git@github.com:spinlock99/haikuter.git HEAD
      01 fc47c6c1741c086808693ebdb85b301c260b681e	HEAD
    ✔ 01 builder@staging.haikuter.com 1.912s
00:02 deploy:check:directories
      01 mkdir -p /var/www/haikuter/shared /var/www/haikuter/releases
    ✔ 01 builder@staging.haikuter.com 0.056s
00:02 asdf:upload_wrapper
      ASDF wrapper already exists on the target host
00:02 asdf:check
      01 /var/www/haikuter/shared/asdf-wrapper asdf current
      01 elixir          1.17.3-otp-27   /home/builder/.tool-versions
      01 erlang          27.1.1          /home/builder/.tool-versions
      01 nodejs          22.9.0          /home/builder/.tool-versions
      01 python          3.12.7          /home/builder/.tool-versions
      01 ruby            3.0.7           /home/builder/.tool-versions
    ✔ 01 builder@staging.haikuter.com 0.335s
00:03 git:clone
      The repository mirror is at /var/www/haikuter/repo
00:03 git:update
      01 git remote set-url origin git@github.com:spinlock99/haikuter.git
    ✔ 01 builder@staging.haikuter.com 0.051s
      02 git remote update --prune
    ✔ 02 builder@staging.haikuter.com 2.032s
00:05 git:create_release
      01 mkdir -p /var/www/haikuter/releases/20241016210416
    ✔ 01 builder@staging.haikuter.com 0.060s
      02 git archive master | /usr/bin/env tar -x -f - -C /var/www/haikuter/releases/20241016…
    ✔ 02 builder@staging.haikuter.com 0.068s
00:06 deploy:set_current_revision
      01 echo "fc47c6c1741c086808693ebdb85b301c260b681e" > REVISION
    ✔ 01 builder@staging.haikuter.com 0.058s
00:06 deploy:set_current_revision_time
      01 echo "1729102279" > REVISION_TIME
    ✔ 01 builder@staging.haikuter.com 0.059s
00:06 deploy:updated
      01 /var/www/haikuter/shared/asdf-wrapper mix deps.get --only prod
      01 * Getting heroicons (https://github.com/tailwindlabs/heroicons.git - v2.1.1)
      01 remote: Enumerating objects: 9296, done.
remote: Counting objects:   1% (93/9…0% (1/9296)        
remote: Counting objects:   7% (65…  6% (558/9296)        
remote: Counting objects:  12% (1…  11% (1023/9296)        
remote: Counting objects:  19% (1…  18% (1674/9296)        
remote: Counting objects:  24% (2…  23% (2139/9296)        
remote: Counting objects:  31% (2…  30% (2789/9296)        
remote: Compressing objects:   1%…ts:   0% (1/5452)        
remote: Compressing objects:  …jects:  98% (5343/5452)        
      01 remote: Total 9296 (delta 4606), reused 7995 (delta 3784), pack-reused 0 (from 0)
      01 Resolving Hex dependencies...
      01 Resolution completed in 0.299s
      01 Unchanged:
      01   bandit 1.5.7
      snip
      01   websock_adapter 0.5.7
      01 * Getting phoenix (Hex package)
      snip
      01 * Getting websock_adapter (Hex package)
    ✔ 01 builder@staging.haikuter.com 38.944s
      02 /var/www/haikuter/shared/asdf-wrapper mix compile
      02 ==> decimal
      02 Compiling 4 files (.ex)
      02 Generated decimal app
      02 ==> mime
      snip
      02 ==> phoenix_ecto
      02 Compiling 7 files (.ex)
      02 Generated phoenix_ecto app
      02 ==> haikuter
      02 Compiling 15 files (.ex)
      02 Generated haikuter app
    ✔ 02 builder@staging.haikuter.com 43.215s
      03 /var/www/haikuter/shared/asdf-wrapper mix assets.deploy
      03
      03 14:05:42.046 [debug] Downloading tailwind from https://github.com/tailwindlabs/tailw…
      03
      03 Rebuilding...
      03
      03 Done in 761ms.
      03
      03 14:09:20.912 [debug] Downloading esbuild from https://registry.npmjs.org/@esbuild/li…
      03
      03   ../priv/static/assets/app.js  112.8kb
      03
      03 ⚡ Done in 23ms
      03 Check your digested files at "priv/static"
    ✔ 03 builder@staging.haikuter.com 232.863s
05:21 deploy:symlink:release
      01 ln -s /var/www/haikuter/releases/20241016210416 /var/www/haikuter/releases/current
    ✔ 01 builder@staging.haikuter.com 0.045s
      02 mv /var/www/haikuter/releases/current /var/www/haikuter
    ✔ 02 builder@staging.haikuter.com 0.049s
05:21 deploy:cleanup
      Keeping 5 of 6 deployed releases on staging.haikuter.com
      01 rm -rf /var/www/haikuter/releases/20241016185208
    ✔ 01 builder@staging.haikuter.com 0.066s
05:21 deploy:log_revision
      01 echo "Branch master (at fc47c6c1741c086808693ebdb85b301c260b681e) deployed as releas…
    ✔ 01 builder@staging.haikuter.com 0.060s
```

> spinlock@Derico:~/src/haikuter$ ssh builder@staging.haikuter.com

```
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-45-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

30 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

5 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Wed Oct 16 12:18:59 2024 from 127.0.0.1
```

> builder@Derico:~$ cd /var/www/haikuter/current
> builder@Derico:/var/www/haikuter/current$ PORT=4001 MIX_ENV=prod mix phx.server

```
14:11:03.432 [info] Running HaikuterWeb.Endpoint with Bandit 1.5.7 at :::4001 (http)
14:11:03.443 [info] Access HaikuterWeb.Endpoint at https://example.com
```

```
namespace :deploy do
  task :updated do
    SSHKit.config.command_map.prefix[:mix].unshift("#{fetch(:asdf_wrapper_path)}")
    on roles(:app) do |host|
      within release_path do
        SSHKit.config.default_env[:MIX_ENV] = 'prod'
        execute(:mix, 'deps.get --only prod')
        execute(:mix, 'compile')
        execute(:mix, 'assets.deploy')
        execute(:mix, 'phx.gen.release')
        execute(:mix, 'release')
      end
    end
  end
end
```

> builder@Derico:/var/www/haikuter/current$ ./_build/prod/rel/haikuter/bin/server

``
14:27:17.559 [info] Running HaikuterWeb.Endpoint with Bandit 1.5.7 at :::4000 (http)
14:27:17.568 [info] Access HaikuterWeb.Endpoint at https://example.com
14:27:29.780 request_id=F_8MESlNRfUwg7YAAAKi [info] GET /
14:27:29.783 request_id=F_8MESlNRfUwg7YAAAKi [info] Sent 200 in 2ms
14:27:37.001 request_id=F_8MEte2_HEwg7YAAALC [info] GET /
14:27:37.003 request_id=F_8MEte2_HEwg7YAAALC [info] Sent 200 in 1ms
14:27:38.851 request_id=F_8ME0Xz_PMwg7YAAALi [info] GET /
14:27:38.852 request_id=F_8ME0Xz_PMwg7YAAALi [info] Sent 200 in 1ms
```
