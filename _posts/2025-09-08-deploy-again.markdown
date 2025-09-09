---
layout: post
title:  "Deploy Again"
date:   2025-09-08 07:00:00 -0700
categories: debian
---
{% include google_analytics.html %}

## Install Nginx

```
spinlock@derico:~/src/haikuter$ sudo apt install nginx-core
```

##  mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied

> spinlock@derico:~/src/haikuter$ cap staging deploy check

```
00:00 git:wrapper
      01 mkdir -p /tmp
    ✔ 01 builder@staging.haikuter.com 0.399s
      Uploading /tmp/git-ssh-a1af82becfbdea207180.sh 100.0%
      02 chmod 700 /tmp/git-ssh-a1af82becfbdea207180.sh
    ✔ 02 builder@staging.haikuter.com 0.055s
00:00 git:check
      01 git ls-remote git@github.com:spinlock99/haikuter.git HEAD
      01 78f9e354b71b32647bcc771a7209712568c47e29	HEAD
    ✔ 01 builder@staging.haikuter.com 3.968s
00:04 deploy:check:directories
      01 mkdir -p /var/www/haikuter/shared /var/www/haikuter/releases
      01 mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
      01 mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
#<Thread:0x000055ea81efb098 /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/runners/parallel.rb:8 run> terminated with exception (report_on_exception is true):
/home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/runners/parallel.rb:13:in `rescue in block (2 levels) in execute': Exception while executing as builder@staging.haikuter.com: mkdir exit status: 1 (SSHKit::Runner::ExecuteError)
mkdir stdout: Nothing written
mkdir stderr: mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
	from /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/runners/parallel.rb:9:in `block (2 levels) in execute'
/home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/command.rb:97:in `exit_status=': mkdir exit status: 1 (SSHKit::Command::Failed)
mkdir stdout: Nothing written
mkdir stderr: mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
	from /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/backends/netssh.rb:185:in `execute_command'
	from /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/backends/abstract.rb:147:in `block in create_command_and_execute'
	from <internal:kernel>:90:in `tap'
	from /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/backends/abstract.rb:147:in `create_command_and_execute'
	from /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/backends/abstract.rb:80:in `execute'
	from /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/capistrano-3.19.2/lib/capistrano/tasks/deploy.rake:67:in `block (4 levels) in <top (required)>'
	from /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/backends/abstract.rb:31:in `instance_exec'
	from /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/backends/abstract.rb:31:in `run'
	from /home/spinlock/.asdf/installs/ruby/3.0.7/lib/ruby/gems/3.0.0/gems/sshkit-1.24.0/lib/sshkit/runners/parallel.rb:10:in `block (2 levels) in execute'
(Backtrace restricted to imported tasks)
cap aborted!
SSHKit::Runner::ExecuteError: Exception while executing as builder@staging.haikuter.com: mkdir exit status: 1
mkdir stdout: Nothing written
mkdir stderr: mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied


Caused by:
SSHKit::Command::Failed: mkdir exit status: 1
mkdir stdout: Nothing written
mkdir stderr: mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied

Tasks: TOP => deploy:check:directories
(See full trace by running task with --trace)
The deploy has failed with an error: Exception while executing as builder@staging.haikuter.com: mkdir exit status: 1
mkdir stdout: Nothing written
mkdir stderr: mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied


** DEPLOY FAILED
** Refer to log/capistrano.log for details. Here are the last 20 lines:


mkdir: cannot create directory ‘/var/www’: Permission denied

  INFO ---------------------------------------------------------------------------

  INFO START 2025-09-08 09:32:45 -0700 cap staging deploy check

  INFO ---------------------------------------------------------------------------

  INFO [fffe5b97] Running /usr/bin/env mkdir -p /tmp as builder@staging.haikuter.com

 DEBUG [fffe5b97] Command: /usr/bin/env mkdir -p /tmp

  INFO [fffe5b97] Finished in 0.399 seconds with exit status 0 (successful).

 DEBUG Uploading /tmp/git-ssh-a1af82becfbdea207180.sh 0.0%

  INFO Uploading /tmp/git-ssh-a1af82becfbdea207180.sh 100.0%

  INFO [9e3e5f63] Running /usr/bin/env chmod 700 /tmp/git-ssh-a1af82becfbdea207180.sh as builder@staging.haikuter.com

 DEBUG [9e3e5f63] Command: /usr/bin/env chmod 700 /tmp/git-ssh-a1af82becfbdea207180.sh

  INFO [9e3e5f63] Finished in 0.055 seconds with exit status 0 (successful).

  INFO [73bf5e5c] Running /usr/bin/env git ls-remote git@github.com:spinlock99/haikuter.git HEAD as builder@staging.haikuter.com

 DEBUG [73bf5e5c] Command: ( export GIT_ASKPASS="/bin/echo" GIT_SSH="/tmp/git-ssh-a1af82becfbdea207180.sh" ; /usr/bin/env git ls-remote git@github.com:spinlock99/haikuter.git HEAD )

 DEBUG [73bf5e5c] 	78f9e354b71b32647bcc771a7209712568c47e29	HEAD

  INFO [73bf5e5c] Finished in 3.968 seconds with exit status 0 (successful).

  INFO [76a1924c] Running /usr/bin/env mkdir -p /var/www/haikuter/shared /var/www/haikuter/releases as builder@staging.haikuter.com

 DEBUG [76a1924c] Command: /usr/bin/env mkdir -p /var/www/haikuter/shared /var/www/haikuter/releases

 DEBUG [76a1924c] 	mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied

mkdir: cannot create directory ‘/var/www/haikuter’: Permission denied
```

> spinlock@derico:~/src/haikuter$ sudo mkdir /var/www/haikuter
> spinlock@derico:~/src/haikuter$ sudo chown builder:builder /var/www/haikuter

## /var/www/haikuter/shared/asdf-wrapper: line 4: exec: asdf: not found

Installed asdf and created a wrapper script to make `capistrano-asdf` happy.
Ultimately, this is a hack and should be fixed in `capistrano-asdf`.

```
builder@derico:~$ cat .asdf/asdf.sh
#!/usr/bin/bash

/home/builder/.asdf/asdf > /dev/null 2>&1

exit
```

Add all of the plugins for the tools:

> builder@derico:/var/www/haikuter/current$ asdf plugin add ruby
> builder@derico:/var/www/haikuter/current$ asdf plugin add elixir
> builder@derico:/var/www/haikuter/current$ asdf plugin add erlang
> builder@derico:/var/www/haikuter/current$ asdf plugin add python
> builder@derico:/var/www/haikuter/current$ asdf plugin add nodejs
> builder@derico:/var/www/haikuter/current$ asdf install

## Now Setup Database

export environment variables from `~/.profile`

## Copy SystemD from Prior Blog Posts

## Copy Nginx Config into sites-available

## HTTPS

Create self signed cert:

```
openssl req -x509 -out localhost.crt -keyout localhost.key \
  -newkey rsa:2048 -nodes -sha256 \
  -subj '/CN=localhost' -extensions EXT -config <( \
   printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```

## ASDF

Add asdf shims to path via .bashrc at the top before it bails as a non-login shell.

## SystemD

- create .system scripts
- `sudo systemctl enable /ets/systemd/system/haikuter.service`


