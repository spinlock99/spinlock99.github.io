---
layout: post
title:  "Debugging Static Assets"
date:   2024-10-11 07:00:00 -0700
categories: elixir phoenix gigalixir
---
{% include google_analytics.html %}

## Ditching Gigalixer

It's just too much work to get it going. I already have a Phoenix Deployment
Pipeline implemented with edeliver and distillery. Mix Releasees have superseeded
distillery releases so I'd like to use them from a stock Phoenix App.

# Mix Release


# Capistrano Deploy

> $ cat .tool-versions

```
ruby 3.0.7
elixir 1.17.3-otp-27
erlang 27.1.1
python 3.12.7
nodejs 22.9.0
```

> $ which gem

```
/home/spinlock/.asdf/shims/gem
```

> $ gem install capistrano

```
Fetching net-scp-4.0.0.gem
Fetching sshkit-1.23.1.gem
Fetching net-sftp-4.0.0.gem
Fetching net-ssh-7.3.0.gem
Fetching capistrano-3.19.1.gem
Fetching airbrussh-1.5.3.gem
Successfully installed net-ssh-7.3.0
Successfully installed net-sftp-4.0.0
Successfully installed net-scp-4.0.0
Successfully installed sshkit-1.23.1
Successfully installed airbrussh-1.5.3
Successfully installed capistrano-3.19.1
Parsing documentation for net-ssh-7.3.0
Installing ri documentation for net-ssh-7.3.0
Parsing documentation for net-sftp-4.0.0
Installing ri documentation for net-sftp-4.0.0
Parsing documentation for net-scp-4.0.0
Installing ri documentation for net-scp-4.0.0
Parsing documentation for sshkit-1.23.1
Installing ri documentation for sshkit-1.23.1
Parsing documentation for airbrussh-1.5.3
Installing ri documentation for airbrussh-1.5.3
Parsing documentation for capistrano-3.19.1
Installing ri documentation for capistrano-3.19.1
Done installing documentation for net-ssh, net-sftp, net-scp, sshkit, airbrussh, capistrano after 3 seconds
6 gems installed
```

> $ ssh builder@app.barronwatchcompany.com

```
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-56-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Oct 14 00:26:08 UTC 2024

  System load:  0.0                Processes:             106
  Usage of /:   57.5% of 24.05GB   Users logged in:       0
  Memory usage: 46%                IPv4 address for eth0: 167.172.115.24
  Swap usage:   0%                 IPv4 address for eth0: 10.46.0.6

135 updates can be applied immediately.
2 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


1 updates could not be installed automatically. For more details,
see /var/log/unattended-upgrades/unattended-upgrades.log

*** System restart required ***
Last login: Sat Sep 17 05:50:55 2022 from 158.51.83.109
```

> builder@bwc-dev:~$ whoami

```
builder
```

> $ ssh builder@staging.haikuter.com 'ls -lR /var/www/haikuter'

```
/var/www/haikuter:
total 0
```


> spinlock@Derico:~/src/haikuter$ cap staging deploy:check --trace

```
** Invoke staging (first_time)
** Execute staging
** Invoke load:defaults (first_time)
** Execute load:defaults
** Invoke deploy:check (first_time)
** Invoke git:check (first_time)
** Invoke git:wrapper (first_time)
** Execute git:wrapper
00:00 git:wrapper
      01 mkdir -p /tmp
    ✔ 01 staging.haikuter.com 0.265s
      Uploading /tmp/git-ssh-188250990dc0a249edb7.sh 100.0%
      02 chmod 700 /tmp/git-ssh-188250990dc0a249edb7.sh
    ✔ 02 staging.haikuter.com 0.063s
** Execute git:check
00:00 git:check
      01 git ls-remote git@github.com:spinlock99/haikuter.git HEAD
      01 064bb9704c9dd8fff1102b08a4d4665434508eae	HEAD
    ✔ 01 staging.haikuter.com 2.162s
** Execute deploy:check
** Invoke deploy:check:directories (first_time)
** Execute deploy:check:directories
00:02 deploy:check:directories
      01 mkdir -p /var/www/haikuter/shared /var/www/haikuter/releases
      01 mkdir: cannot create directory ‘/var/www/haikuter/shared’
      01 : Permission denied
      01 mkdir: cannot create directory ‘/var/www/haikuter/releases’: Permission denied
#<Thread:0x0000635eb3169408 /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.23.1/lib/sshkit/runners/parallel.rb:10 run> terminated with exception (report_on_exception is true):
/home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.23.1/lib/sshkit/runners/parallel.rb:15:in `rescue in block (2 levels) in execute': Exception while executing on host staging.haikuter.com: mkdir exit status: 1 (SSHKit::Runner::ExecuteError)
mkdir stdout: Nothing written
mkdir stderr: mkdir: cannot create directory ‘/var/www/haikuter/shared’: Permission denied
mkdir: cannot create directory ‘/var/www/haikuter/releases’: Permission denied
```

> $ vi config/deploy/staging.rb

'''
# role-based syntax
# ==================

 role :web, %w{builder@staging.haikuter.com}
'''

> spinlock@Derico:~/src/haikuter$ cap staging deploy:check

```
00:00 git:wrapper
      01 mkdir -p /tmp
    ✔ 01 builder@staging.haikuter.com 0.648s
      Uploading /tmp/git-ssh-d4b50ac5e190f3dbb68f.sh 100.0%
      02 chmod 700 /tmp/git-ssh-d4b50ac5e190f3dbb68f.sh
    ✔ 02 builder@staging.haikuter.com 0.045s
00:00 git:check
      01 git ls-remote git@github.com:spinlock99/haikuter.git HEAD
      01 Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
      01 064bb9704c9dd8fff1102b08a4d4665434508eae	HEAD
    ✔ 01 builder@staging.haikuter.com 2.691s
00:03 deploy:check:directories
      01 mkdir -p /var/www/haikuter/shared /var/www/haikuter/releases
    ✔ 01 builder@staging.haikuter.com 0.048s
```

> spinlock@Derico:~/src/haikuter$ cap staging deploy

```
00:00 git:wrapper
      01 mkdir -p /tmp
verify_host_key: :secure is deprecated, use :always
    ✔ 01 builder@staging.haikuter.com 0.636s
      Uploading /tmp/git-ssh-b6c66e788e1a40a04bcb.sh 100.0%
      02 chmod 700 /tmp/git-ssh-b6c66e788e1a40a04bcb.sh
    ✔ 02 builder@staging.haikuter.com 0.048s
00:00 git:check
      01 git ls-remote git@github.com:spinlock99/haikuter.git HEAD
      01 064bb9704c9dd8fff1102b08a4d4665434508eae	HEAD
    ✔ 01 builder@staging.haikuter.com 2.495s
00:03 deploy:check:directories
      01 mkdir -p /var/www/haikuter/shared /var/www/haikuter/releases
    ✔ 01 builder@staging.haikuter.com 0.056s
00:03 git:clone
      01 git clone --mirror git@github.com:spinlock99/haikuter.git /var/www/haikuter/repo
      01 Cloning into bare repository '/var/www/haikuter/repo'...
    ✔ 01 builder@staging.haikuter.com 4.256s
00:07 git:update
      01 git remote set-url origin git@github.com:spinlock99/haikuter.git
    ✔ 01 builder@staging.haikuter.com 0.062s
      02 git remote update --prune
    ✔ 02 builder@staging.haikuter.com 1.921s
00:09 git:create_release
      01 mkdir -p /var/www/haikuter/releases/20241015191637
    ✔ 01 builder@staging.haikuter.com 0.058s
      02 git archive master | /usr/bin/env tar -x -f - -C /var/www/haikuter/releases/20241015…
    ✔ 02 builder@staging.haikuter.com 0.066s
00:10 deploy:set_current_revision
      01 echo "064bb9704c9dd8fff1102b08a4d4665434508eae" > REVISION
    ✔ 01 builder@staging.haikuter.com 0.056s
00:10 deploy:set_current_revision_time
      01 echo "1728866711" > REVISION_TIME
    ✔ 01 builder@staging.haikuter.com 0.059s
00:10 deploy:symlink:release
      01 ln -s /var/www/haikuter/releases/20241015191637 /var/www/haikuter/releases/current
    ✔ 01 builder@staging.haikuter.com 0.046s
      02 mv /var/www/haikuter/releases/current /var/www/haikuter
    ✔ 02 builder@staging.haikuter.com 0.047s
00:10 deploy:log_revision
      01 echo "Branch master (at 064bb9704c9dd8fff1102b08a4d4665434508eae) deployed as releas…
    ✔ 01 builder@staging.haikuter.com 0.057s
```

> spinlock@Derico:~/src/haikuter$ ls /var/www/haikuter/

```
current  releases  repo  revisions.log  shared
```

> spinlock@Derico:~/src/haikuter$ ls /var/www/haikuter/current

```
assets   config  mix.exs   priv       REVISION       test
Capfile  lib     mix.lock  README.md  REVISION_TIME
```
