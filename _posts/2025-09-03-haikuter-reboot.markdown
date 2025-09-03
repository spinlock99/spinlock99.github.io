---
layout: post
title:  "New Laptop: HP ProBook"
date:   2025-07-22 07:00:00 -0700
categories: debian
---
{% include google_analytics.html %}

## Install Ruby

We're using `ASDF` to manage our tools so the required version to build and
deploy `Haikuter` are in `.tool-versions`:

```
spinlock@derico:~/src/haikuter$ vi .tool-versions
```

Install `Ruby` so we can run `Capistrano`.

```
spinlock@derico:~/src/haikuter$ asdf plugin add ruby
spinlock@derico:~/src/haikuter$ asdf install ruby
Downloading ruby-build...
==> Downloading openssl-1.1.1w.tar.gz...
-> curl -q -fL -o openssl-1.1.1w.tar.gz https://dqw8nmjcqpjn7.cloudfront.net/cf3098950cb4d853ad95c0841f1f9c6d3dc102dccfcacd521d93925208b76ac8
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 9661k  100 9661k    0     0  9013k      0  0:00:01  0:00:01 --:--:-- 9021k
==> Installing openssl-1.1.1w...
/home/spinlock/.asdf/plugins/ruby/ruby-build/bin/ruby-build: line 725: [: : integer expression expected
-> ./config "--prefix=$HOME/.asdf/installs/ruby/3.0.7/openssl" "--openssldir=$HOME/.asdf/installs/ruby/3.0.7/openssl/ssl" --libdir=lib zlib-dynamic no-ssl3 shared "-Wl,-rpath,$HOME/.asdf/installs/ruby/3.0.7/openssl/lib"
-> make -j 16
-> make install_sw install_ssldirs
==> Installed openssl-1.1.1w to /home/spinlock/.asdf/installs/ruby/3.0.7
==> Downloading ruby-3.0.7.tar.gz...
-> curl -q -fL -o ruby-3.0.7.tar.gz https://cache.ruby-lang.org/pub/ruby/3.0/ruby-3.0.7.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 20.2M  100 20.2M    0     0  5002k      0  0:00:04  0:00:04 --:--:-- 5065k
==> Installing ruby-3.0.7...

WARNING: ruby-3.0.7 is past its end of life and is now unsupported.
It no longer receives bug fixes or critical security updates.

-> ./configure "--prefix=$HOME/.asdf/installs/ruby/3.0.7" "--with-openssl-dir=$HOME/.asdf/installs/ruby/3.0.7/openssl" --enable-shared --with-ext=openssl,psych,+
-> make -j 16
-> make install
==> Installed ruby-3.0.7 to /home/spinlock/.asdf/installs/ruby/3.0.7
spinlock@derico:~/src/haikuter$
```

### Ruby is Borked

```
spinlock@derico:~/src/haikuter$ bundle exec cap -T
Could not locate Gemfile or .bundle/ directory
```


### Missing Gemfile

```
spinlock@derico:~/src/haikuter$ bundle init
Writing new Gemfile to /home/spinlock/src/haikuter/Gemfile
spinlock@derico:~/src/haikuter$ vi Gemfile
spinlock@derico:~/src/haikuter$ bundle install
Fetching gem metadata from https://rubygems.org/.......
Resolving dependencies...
Using rake 13.3.0
Fetching base64 0.3.0
Using net-ssh 7.3.0
Using bundler 2.2.33
Using net-sftp 4.0.0
Fetching ostruct 0.6.3
Fetching logger 1.7.0
Using concurrent-ruby 1.3.5
Using i18n 1.14.7
Using net-scp 4.1.0
Installing base64 0.3.0
Installing ostruct 0.6.3
Installing logger 1.7.0
Using sshkit 1.24.0
Using airbrussh 1.5.3
Using capistrano 3.19.2
Bundle complete! 1 Gemfile dependency, 13 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

```
spinlock@derico:~/src/haikuter$ cap staging deploy --dry-run
(Backtrace restricted to imported tasks)
cap aborted!
LoadError: cannot load such file -- capistrano/asdf
/home/spinlock/src/haikuter/Capfile:36:in `<top (required)>'
(See full trace by running task with --trace)
```

## Success!!!

Add `capistrano-asdf` to Gemfile:

```
spinlock@derico:~/src/haikuter$ vi Gemfile
```
```
1 # frozen_string_literal: true
2
3 source "https://rubygems.org"
4
5 git_source(:github) { |repo_name| "https://github.com/#{repo_name}" }
6
7 # gem "rails"
8 gem "capistrano", require: false
9 gem "capistrano-asdf"
```

```
spinlock@derico:~/src/haikuter$ bundle install
Fetching gem metadata from https://rubygems.org/......
Resolving dependencies...
Using rake 13.3.0
Using bundler 2.2.33
Using concurrent-ruby 1.3.5
Using logger 1.7.0
Using net-ssh 7.3.0
Using ostruct 0.6.3
Using base64 0.3.0
Using net-sftp 4.0.0
Using i18n 1.14.7
Using net-scp 4.1.0
Using sshkit 1.24.0
Using airbrussh 1.5.3
Using capistrano 3.19.2
Fetching capistrano-asdf 1.1.1
Installing capistrano-asdf 1.1.1
Bundle complete! 2 Gemfile dependencies, 14 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
spinlock@derico:~/src/haikuter$ cap staging deploy --dry-run
00:00 git:wrapper
      01 mkdir -p /tmp
      02 #<StringIO:0x000055de6c85a098> /tmp/git-ssh-14f2fcfd209557a6971a.sh
      03 chmod 700 /tmp/git-ssh-14f2fcfd209557a6971a.sh
00:00 git:check
      01 git ls-remote git@github.com:spinlock99/haikuter.git HEAD
00:00 deploy:check:directories
      01 mkdir -p /var/www/haikuter/shared /var/www/haikuter/releases
00:00 asdf:upload_wrapper
      ASDF wrapper already exists on the target host but is different from the one we want to upload
      01 #<StringIO:0x000055de6cec3240> /var/www/haikuter/shared/asdf-wrapper
      02 chmod +x /var/www/haikuter/shared/asdf-wrapper
00:00 asdf:check
      01 /var/www/haikuter/shared/asdf-wrapper asdf current
00:00 git:clone
      The repository mirror is at /var/www/haikuter/repo
00:00 git:update
      01 git remote set-url origin git@github.com:spinlock99/haikuter.git
      02 git remote update --prune
00:00 git:create_release
      01 mkdir -p /var/www/haikuter/releases/20250903164616
      02 git archive master | /usr/bin/env tar -x -f - -C /var/www/haikuter/releases/20250903164616
00:00 deploy:set_current_revision
      01 echo "" > REVISION
00:00 deploy:set_current_revision_time
      01 echo "" > REVISION_TIME
00:00 deploy:build_release
      01 /var/www/haikuter/shared/asdf-wrapper mix deps.get --only prod >/dev/null
      02 /var/www/haikuter/shared/asdf-wrapper mix compile >/dev/null
      03 /var/www/haikuter/shared/asdf-wrapper mix assets.deploy >/dev/null
      04 /var/www/haikuter/shared/asdf-wrapper mix phx.gen.release >/dev/null
      05 /var/www/haikuter/shared/asdf-wrapper mix release >/dev/null
00:00 deploy:symlink:release
      01 ln -s /var/www/haikuter/releases/20250903164616 /var/www/haikuter/releases/current
      02 mv /var/www/haikuter/releases/current /var/www/haikuter
00:00 deploy:symlink:restart_service
      01 touch /var/www/haikuter/current
00:00 deploy:log_revision
      01 echo "Branch master (at ) deployed as release 20250903164616 by spinlock" >> /var/www/haikuter/revisions.log
spinlock@derico:~/src/haikuter$

```
