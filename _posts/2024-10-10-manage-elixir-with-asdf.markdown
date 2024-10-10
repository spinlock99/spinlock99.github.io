---
layout: post
title:  "Managing Elixir with ASDF"
date:   2024-10-10 07:00:00 -0700
categories: elixir
---
{% include google_analytics.html %}

> Install Plugins

```
$ asdf plugin add erlang
updating plugin repository...HEAD is now at 00af419 feat: adding plugin for avalanchego (#1058)
$ asdf plugin add elixir
```

> Install Erlang/OTP

```
$ asdf list-all erlang
...
27.0.1
27.1
27.1.1
$ asdf install erlang 27.1.1
ERROR: 'asdf_27.1.1' is not a kerl-managed Erlang/OTP installation.
Build 'asdf_27.1.1' has been deleted.
/home/spinlock/.asdf/downloads/erlang/27.1.1/OTP-27.1.1.tar.gz (from GitHub) corrupted; re-downloading...
Extracting source code for normal build...
Building (normal) Erlang/OTP 27.1.1 (asdf_27.1.1); please wait...
Initializing (build) log file at /home/spinlock/.asdf/plugins/erlang/kerl-home/builds/asdf_27.1.1/otp_build_27.1.1.log.
WARNING: [packages] Probe failed for libncurses-dev (distro: ubuntu): probe "dpkg-query -Wf'${db:Status-abbrev}' "libncurses-dev" 2>/dev/null | \grep -q '^i'" returned 1
ERROR: configure failed.
checking for kstat_open in -lkstat... (cached) no
checking for tgetent in -ltinfo... no
checking for tgetent in -lncurses... no
checking for tgetent in -lcurses... no
checking for tgetent in -ltermcap... no
checking for tgetent in -ltermlib... no
configure: error: No curses library functions found
ERROR: /home/spinlock/.asdf/plugins/erlang/kerl-home/builds/asdf_27.1.1/otp_src_27.1.1/erts/configure failed!
./configure: 370: kill: No such process


Please see /home/spinlock/.asdf/plugins/erlang/kerl-home/builds/asdf_27.1.1/otp_build_27.1.1.log for full details.
Auto cleaning all artifacts except the log file...
(use KERL_AUTOCLEAN=0 to keep build on failure, if desired)
Cleaning up compilation products for 'asdf_27.1.1' under:
  - /home/spinlock/.asdf/plugins/erlang/kerl-home/builds...
  - /home/spinlock/.asdf/downloads/erlang/27.1.1...
... done.
```

```
spinlock@Derico:~/src/spinlock99.github.io$ asdf list-all elixir
0.12.4
0.12.5
0.13.0
0.13.1
0.13.2
0.13.3
0.14.0
0.14.1
0.14.2
0.14.3
0.15.0
0.15.1
1.0.0
1.0.0-otp-17
1.0.0-rc1
1.0.0-rc1-otp-17
1.0.0-rc2
1.0.0-rc2-otp-17
1.0.1
1.0.1-otp-17
1.0.2
1.0.2-otp-17
1.0.3
1.0.3-otp-17
1.0.4
1.0.4-otp-17
1.0.5
1.0.5-otp-17
1.0.5-otp-18
1.1.0
1.1.0-otp-17
1.1.0-otp-18
1.1.0-rc.0
1.1.0-rc.0-otp-17
1.1.0-rc.0-otp-18
1.1.1
1.1.1-otp-17
1.1.1-otp-18
1.2.0
1.2.0-otp-18
1.2.0-rc.0
1.2.0-rc.0-otp-18
1.2.0-rc.1
1.2.0-rc.1-otp-18
1.2.1
1.2.1-otp-18
1.2.2
1.2.2-otp-18
1.2.3
1.2.3-otp-18
1.2.4
1.2.4-otp-18
1.2.5
1.2.5-otp-18
1.2.6
1.2.6-otp-18
1.2.6-otp-19
1.3.0
1.3.0-otp-18
1.3.0-otp-19
1.3.0-rc.0
1.3.0-rc.0-otp-18
1.3.0-rc.0-otp-19
1.3.0-rc.1
1.3.0-rc.1-otp-18
1.3.0-rc.1-otp-19
1.3.1
1.3.1-otp-18
1.3.1-otp-19
1.3.2
1.3.2-otp-18
1.3.2-otp-19
1.3.3
1.3.3-otp-18
1.3.3-otp-19
1.3.4
1.3.4-otp-18
1.3.4-otp-19
1.4.0
1.4.0-otp-18
1.4.0-otp-19
1.4.0-rc.0
1.4.0-rc.0-otp-18
1.4.0-rc.0-otp-19
1.4.0-rc.0-otp-20
1.4.0-rc.1
1.4.0-rc.1-otp-18
1.4.0-rc.1-otp-19
1.4.0-rc.1-otp-20
1.4.1
1.4.1-otp-18
1.4.1-otp-19
1.4.2
1.4.2-otp-18
1.4.2-otp-19
1.4.3
1.4.3-otp-18
1.4.3-otp-19
1.4.4
1.4.4-otp-18
1.4.4-otp-19
1.4.5
1.4.5-otp-18
1.4.5-otp-19
1.4.5-otp-20
1.5.0
1.5.0-otp-18
1.5.0-otp-19
1.5.0-otp-20
1.5.0-rc.0
1.5.0-rc.0-otp-18
1.5.0-rc.0-otp-19
1.5.0-rc.0-otp-20
1.5.0-rc.1
1.5.0-rc.1-otp-18
1.5.0-rc.1-otp-19
1.5.0-rc.1-otp-20
1.5.0-rc.2
1.5.0-rc.2-otp-18
1.5.0-rc.2-otp-19
1.5.0-rc.2-otp-20
1.5.1
1.5.1-otp-18
1.5.1-otp-19
1.5.1-otp-20
1.5.2
1.5.2-otp-18
1.5.2-otp-19
1.5.2-otp-20
1.5.3
1.5.3-otp-18
1.5.3-otp-19
1.5.3-otp-20
1.6.0
1.6.0-otp-19
1.6.0-otp-20
1.6.0-rc.0
1.6.0-rc.0-otp-19
1.6.0-rc.0-otp-20
1.6.0-rc.1
1.6.0-rc.1-otp-19
1.6.0-rc.1-otp-20
1.6.1
1.6.1-otp-19
1.6.1-otp-20
1.6.2
1.6.2-otp-19
1.6.2-otp-20
1.6.3
1.6.3-otp-19
1.6.3-otp-20
1.6.4
1.6.4-otp-19
1.6.4-otp-20
1.6.5
1.6.5-otp-19
1.6.5-otp-20
1.6.5-otp-21
1.6.6
1.6.6-otp-19
1.6.6-otp-20
1.6.6-otp-21
1.7.0
1.7.0-otp-19
1.7.0-otp-20
1.7.0-otp-21
1.7.0-otp-22
1.7.0-rc.0
1.7.0-rc.0-otp-19
1.7.0-rc.0-otp-20
1.7.0-rc.0-otp-21
1.7.0-rc.0-otp-22
1.7.0-rc.1
1.7.0-rc.1-otp-19
1.7.0-rc.1-otp-20
1.7.0-rc.1-otp-21
1.7.0-rc.1-otp-22
1.7.1
1.7.1-otp-19
1.7.1-otp-20
1.7.1-otp-21
1.7.1-otp-22
1.7.2
1.7.2-otp-19
1.7.2-otp-20
1.7.2-otp-21
1.7.2-otp-22
1.7.3
1.7.3-otp-19
1.7.3-otp-20
1.7.3-otp-21
1.7.3-otp-22
1.7.4
1.7.4-otp-19
1.7.4-otp-20
1.7.4-otp-21
1.7.4-otp-22
1.8.0
1.8.0-otp-20
1.8.0-otp-21
1.8.0-otp-22
1.8.0-rc.0
1.8.0-rc.0-otp-20
1.8.0-rc.0-otp-21
1.8.0-rc.0-otp-22
1.8.0-rc.1
1.8.0-rc.1-otp-20
1.8.0-rc.1-otp-21
1.8.0-rc.1-otp-22
1.8.1
1.8.1-otp-20
1.8.1-otp-21
1.8.1-otp-22
1.8.2
1.8.2-otp-20
1.8.2-otp-21
1.8.2-otp-22
1.9.0
1.9.0-otp-20
1.9.0-otp-21
1.9.0-otp-22
1.9.0-rc.0
1.9.0-rc.0-otp-20
1.9.0-rc.0-otp-21
1.9.0-rc.0-otp-22
1.9.1
1.9.1-otp-20
1.9.1-otp-21
1.9.1-otp-22
1.9.2
1.9.2-otp-20
1.9.2-otp-21
1.9.2-otp-22
1.9.3
1.9.3-otp-20
1.9.3-otp-21
1.9.3-otp-22
1.9.4
1.9.4-otp-20
1.9.4-otp-21
1.9.4-otp-22
1.10.0
1.10.0-otp-21
1.10.0-otp-22
1.10.0-rc.0
1.10.0-rc.0-otp-21
1.10.0-rc.0-otp-22
1.10.1
1.10.1-otp-21
1.10.1-otp-22
1.10.2
1.10.2-otp-21
1.10.2-otp-22
1.10.3
1.10.3-otp-21
1.10.3-otp-22
1.10.3-otp-23
1.10.4
1.10.4-otp-21
1.10.4-otp-22
1.10.4-otp-23
1.11.0
1.11.0-otp-21
1.11.0-otp-22
1.11.0-otp-23
1.11.0-rc.0
1.11.0-rc.0-otp-21
1.11.0-rc.0-otp-22
1.11.0-rc.0-otp-23
1.11.1
1.11.1-otp-21
1.11.1-otp-22
1.11.1-otp-23
1.11.2
1.11.2-otp-21
1.11.2-otp-22
1.11.2-otp-23
1.11.3
1.11.3-otp-21
1.11.3-otp-22
1.11.3-otp-23
1.11.4
1.11.4-otp-21
1.11.4-otp-22
1.11.4-otp-23
1.11.4-otp-24
1.12.0
1.12.0-otp-22
1.12.0-otp-23
1.12.0-otp-24
1.12.0-rc.0
1.12.0-rc.0-otp-21
1.12.0-rc.0-otp-22
1.12.0-rc.0-otp-23
1.12.0-rc.0-otp-24
1.12.0-rc.1
1.12.0-rc.1-otp-22
1.12.0-rc.1-otp-23
1.12.0-rc.1-otp-24
1.12.1
1.12.1-otp-22
1.12.1-otp-23
1.12.1-otp-24
1.12.2
1.12.2-otp-22
1.12.2-otp-23
1.12.2-otp-24
1.12.3
1.12.3-otp-22
1.12.3-otp-23
1.12.3-otp-24
1.13.0
1.13.0-otp-22
1.13.0-otp-23
1.13.0-otp-24
1.13.0-otp-25
1.13.0-rc.0
1.13.0-rc.0-otp-22
1.13.0-rc.0-otp-23
1.13.0-rc.0-otp-24
1.13.0-rc.0-otp-25
1.13.0-rc.1
1.13.0-rc.1-otp-22
1.13.0-rc.1-otp-23
1.13.0-rc.1-otp-24
1.13.0-rc.1-otp-25
1.13.1
1.13.1-otp-22
1.13.1-otp-23
1.13.1-otp-24
1.13.1-otp-25
1.13.2
1.13.2-otp-22
1.13.2-otp-23
1.13.2-otp-24
1.13.2-otp-25
1.13.3
1.13.3-otp-22
1.13.3-otp-23
1.13.3-otp-24
1.13.3-otp-25
1.13.4
1.13.4-otp-22
1.13.4-otp-23
1.13.4-otp-24
1.13.4-otp-25
1.14.0
1.14.0-otp-23
1.14.0-otp-24
1.14.0-otp-25
1.14.0-rc.0
1.14.0-rc.0-otp-23
1.14.0-rc.0-otp-24
1.14.0-rc.0-otp-25
1.14.0-rc.1
1.14.0-rc.1-otp-23
1.14.0-rc.1-otp-24
1.14.0-rc.1-otp-25
1.14.1
1.14.1-otp-23
1.14.1-otp-24
1.14.1-otp-25
1.14.2
1.14.2-otp-23
1.14.2-otp-24
1.14.2-otp-25
1.14.3
1.14.3-otp-23
1.14.3-otp-24
1.14.3-otp-25
1.14.4
1.14.4-otp-23
1.14.4-otp-24
1.14.4-otp-25
1.14.4-otp-26
1.14.5
1.14.5-otp-23
1.14.5-otp-24
1.14.5-otp-25
1.14.5-otp-26
1.15.0
1.15.0-otp-24
1.15.0-otp-25
1.15.0-otp-26
1.15.0-rc.0
1.15.0-rc.0-otp-24
1.15.0-rc.0-otp-25
1.15.0-rc.0-otp-26
1.15.0-rc.1
1.15.0-rc.1-otp-24
1.15.0-rc.1-otp-25
1.15.0-rc.1-otp-26
1.15.0-rc.2
1.15.0-rc.2-otp-24
1.15.0-rc.2-otp-25
1.15.0-rc.2-otp-26
1.15.1
1.15.1-otp-24
1.15.1-otp-25
1.15.1-otp-26
1.15.2
1.15.2-otp-24
1.15.2-otp-25
1.15.2-otp-26
1.15.3
1.15.3-otp-24
1.15.3-otp-25
1.15.3-otp-26
1.15.4
1.15.4-otp-24
1.15.4-otp-25
1.15.4-otp-26
1.15.5
1.15.5-otp-24
1.15.5-otp-25
1.15.5-otp-26
1.15.6
1.15.6-otp-24
1.15.6-otp-25
1.15.6-otp-26
1.15.7
1.15.7-otp-24
1.15.7-otp-25
1.15.7-otp-26
1.15.8
1.15.8-otp-24
1.15.8-otp-25
1.15.8-otp-26
1.16.0
1.16.0-otp-24
1.16.0-otp-25
1.16.0-otp-26
1.16.0-rc.0
1.16.0-rc.0-otp-24
1.16.0-rc.0-otp-25
1.16.0-rc.0-otp-26
1.16.0-rc.1
1.16.0-rc.1-otp-24
1.16.0-rc.1-otp-25
1.16.0-rc.1-otp-26
1.16.1
1.16.1-otp-24
1.16.1-otp-25
1.16.1-otp-26
1.16.2
1.16.2-otp-24
1.16.2-otp-25
1.16.2-otp-26
1.16.3
1.16.3-otp-24
1.16.3-otp-25
1.16.3-otp-26
1.17.0
1.17.0-otp-25
1.17.0-otp-26
1.17.0-otp-27
1.17.0-rc.0
1.17.0-rc.0-otp-25
1.17.0-rc.0-otp-26
1.17.0-rc.0-otp-27
1.17.0-rc.1
1.17.0-rc.1-otp-25
1.17.0-rc.1-otp-26
1.17.0-rc.1-otp-27
1.17.1
1.17.1-otp-25
1.17.1-otp-26
1.17.1-otp-27
1.17.2
1.17.2-otp-25
1.17.2-otp-26
1.17.2-otp-27
1.17.3
1.17.3-otp-25
1.17.3-otp-26
1.17.3-otp-27
main
main-otp-22
main-otp-23
main-otp-24
main-otp-25
main-otp-26
main-otp-27
master
master-otp-21
master-otp-22
master-otp-23
master-otp-24
spinlock@Derico:~/src/spinlock99.github.io$ asdf install elixir 1.17.3-otp-27
==> Checking whether specified Elixir release exists...
==> Downloading 1.17.3-otp-27 to /home/spinlock/.asdf/downloads/elixir/1.17.3-otp-27/elixir-precompiled-1.17.3-otp-27.zip
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 7196k  100 7196k    0     0   673k      0  0:00:10  0:00:10 --:--:--  530k
==> Copying release into place
```

> Add NCurses Development Libraries

```
spinlock@Derico:~/src$ sudo apt install ncurses-dev
[sudo] password for spinlock: 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Note, selecting 'libncurses-dev' instead of 'ncurses-dev'
Suggested packages:
  ncurses-doc
The following NEW packages will be installed:
  libncurses-dev
0 upgraded, 1 newly installed, 0 to remove and 3 not upgraded.
Need to get 384 kB of archives.
After this operation, 2,417 kB of additional disk space will be used.
Get:1 http://us.archive.ubuntu.com/ubuntu noble/main amd64 libncurses-dev amd64 6.4+20240113-1ubuntu2 [384 kB]
Fetched 384 kB in 6s (60.0 kB/s)                                                             
Selecting previously unselected package libncurses-dev:amd64.
(Reading database ... 231389 files and directories currently installed.)
Preparing to unpack .../libncurses-dev_6.4+20240113-1ubuntu2_amd64.deb ...
Unpacking libncurses-dev:amd64 (6.4+20240113-1ubuntu2) ...
Setting up libncurses-dev:amd64 (6.4+20240113-1ubuntu2) ...
Processing triggers for man-db (2.12.0-4build2) ...
spinlock@Derico:~/src$ asdf install erlang latest
ERROR: 'asdf_27.1.1' is not a kerl-managed Erlang/OTP installation.
Build 'asdf_27.1.1' has been deleted.
Extracting source code for normal build...
Building (normal) Erlang/OTP 27.1.1 (asdf_27.1.1); please wait...
Initializing (build) log file at /home/spinlock/.asdf/plugins/erlang/kerl-home/builds/asdf_27.1.1/otp_build_27.1.1.log.
APPLICATIONS DISABLED (See: /home/spinlock/.asdf/plugins/erlang/kerl-home/builds/asdf_27.1.1/otp_build_27.1.1.log)
 * jinterface     : No Java compiler found
 * odbc           : ODBC library - link check failed

APPLICATIONS INFORMATION (See: /home/spinlock/.asdf/plugins/erlang/kerl-home/builds/asdf_27.1.1/otp_build_27.1.1.log)
 * wx             : No OpenGL headers found, wx will NOT be usable
 * No GLU headers found, wx will NOT be usable
 * wxWidgets was not compiled with --enable-webview or wxWebView developer package is not installed, wxWebView will NOT be available
 *         wxWidgets must be installed on your system.
 *         Please check that wx-config is in path, the directory
 *         where wxWidgets libraries are installed (returned by
 *         'wx-config --libs' or 'wx-config --static --libs' command)
 *         is in LD_LIBRARY_PATH or equivalent variable and
 *         wxWidgets version is 3.0.2 or above.

Erlang/OTP 27.1.1 (asdf_27.1.1) has been successfully built.
Cleaning up compilation products for 'asdf_27.1.1' under:
  - /home/spinlock/.asdf/plugins/erlang/kerl-home/builds...
  - /home/spinlock/.asdf/downloads/erlang/27.1.1...
... done.
spinlock@Derico:~/src$ asdf global erlang latest
```

> Install Phoenix

```
spinlock@Derico:~/src$ mix archive.install hex phx_new
Resolving Hex dependencies...
Resolution completed in 0.066s
New:
  phx_new 1.7.14
* Getting phx_new (Hex package)
All dependencies are up to date
Compiling 11 files (.ex)
Generated phx_new app
Generated archive "phx_new-1.7.14.ez" with MIX_ENV=prod
Found existing entry: /home/spinlock/.asdf/installs/elixir/1.17.3-otp-27/.mix/archives/phx_new
Are you sure you want to replace it with "phx_new-1.7.14.ez"? [Yn]
* creating /home/spinlock/.asdf/installs/elixir/1.17.3-otp-27/.mix/archives/phx_new-1.7.14
spinlock@Derico:~/src$ mix local.phx
Resolving Hex dependencies...
Resolution completed in 0.045s
New:
  phx_new 1.7.14
* Getting phx_new (Hex package)
All dependencies are up to date
Compiling 11 files (.ex)
    warning: redefining module Mix.Tasks.Local.Phx (current version loaded from /home/spinlock/.asdf/installs/elixir/1.17.3-otp-27/.mix/archives/phx_new-1.7.14/phx_new-1.7.14/ebin/Elixir.Mix.Tasks.Local.Phx.beam)
    │
  1 │ defmodule Mix.Tasks.Local.Phx do
    │ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    │
    └─ lib/mix/tasks/local.phx.ex:1: Mix.Tasks.Local.Phx (module)

Generated phx_new app
Generated archive "phx_new-1.7.14.ez" with MIX_ENV=prod
Found existing entry: /home/spinlock/.asdf/installs/elixir/1.17.3-otp-27/.mix/archives/phx_new-1.7.14
Are you sure you want to replace it with "phx_new-1.7.14.ez"? [Yn]
* creating /home/spinlock/.asdf/installs/elixir/1.17.3-otp-27/.mix/archives/phx_new-1.7.14
spinlock@Derico:~/src$ mix phx.new blog
* creating blog/lib/blog/application.ex
* creating blog/lib/blog.ex
* creating blog/lib/blog_web/controllers/error_json.ex
* creating blog/lib/blog_web/endpoint.ex
* creating blog/lib/blog_web/router.ex
* creating blog/lib/blog_web/telemetry.ex
* creating blog/lib/blog_web.ex
* creating blog/mix.exs
* creating blog/README.md
* creating blog/.formatter.exs
* creating blog/.gitignore
* creating blog/test/support/conn_case.ex
* creating blog/test/test_helper.exs
* creating blog/test/blog_web/controllers/error_json_test.exs
* creating blog/lib/blog/repo.ex
* creating blog/priv/repo/migrations/.formatter.exs
* creating blog/priv/repo/seeds.exs
* creating blog/test/support/data_case.ex
* creating blog/lib/blog_web/controllers/error_html.ex
* creating blog/test/blog_web/controllers/error_html_test.exs
* creating blog/lib/blog_web/components/core_components.ex
* creating blog/lib/blog_web/controllers/page_controller.ex
* creating blog/lib/blog_web/controllers/page_html.ex
* creating blog/lib/blog_web/controllers/page_html/home.html.heex
* creating blog/test/blog_web/controllers/page_controller_test.exs
* creating blog/lib/blog_web/components/layouts/root.html.heex
* creating blog/lib/blog_web/components/layouts/app.html.heex
* creating blog/lib/blog_web/components/layouts.ex
* creating blog/priv/static/images/logo.svg
* creating blog/lib/blog/mailer.ex
* creating blog/lib/blog_web/gettext.ex
* creating blog/priv/gettext/en/LC_MESSAGES/errors.po
* creating blog/priv/gettext/errors.pot
* creating blog/priv/static/robots.txt
* creating blog/priv/static/favicon.ico
* creating blog/assets/js/app.js
* creating blog/assets/vendor/topbar.js
* creating blog/assets/css/app.css
* creating blog/assets/tailwind.config.js

Fetch and install dependencies? [Yn]
* running mix deps.get
* running mix assets.setup
* running mix deps.compile

We are almost there! The following steps are missing:

    $ cd blog

Then configure your database in config/dev.exs and run:

    $ mix ecto.create

Start your Phoenix app with:

    $ mix phx.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phx.server

```
