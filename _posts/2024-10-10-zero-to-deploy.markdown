---
layout: post
title:  "Zero to Deploy"
date:   2024-10-10 16:24:00 -0700
categories: elixir
---
{% include google_analytics.html %}

## Zero

[Implement Tutorial](https://thephoenixtutorial.org/book/ch1_from_zero_to_deploy)

## Create Github Repo

# Push local repo upstream

> $ git remote add origin git@github.com:spinlock99/blog.git
> $ git push -u origin master

```
ERROR: Repository not found.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

# Create Repo via www.github.com

> $ git push -u origin master

```
Enumerating objects: 102, done.
Counting objects: 100% (102/102), done.
Delta compression using up to 4 threads
Compressing objects: 100% (89/89), done.
Writing objects: 100% (102/102), 46.64 KiB | 2.03 MiB/s, done.
Total 102 (delta 8), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (8/8), done.
To github.com:spinlock99/blog.git
 * [new branch]      master -> master
branch 'master' set up to track 'origin/master'.
```

> $ git status

```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

## Gigalixir: Elixir Platform as a Service

https://www.gigalixir.com/docs/

```
pip3 install gigalixir --user
echo "export PATH=\$PATH:$(python3 -m site --user-base)/bin" >> ~/.profile
source ~/.profile
```

```
spinlock@Derico:~/src/dotfiles$ pip3 install gigalixir --user
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.

    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.

    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.

    See /usr/share/doc/python3.12/README.venv for more information.

note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
hint: See PEP 668 for the detailed specification.
```

# Manage Python in ASDF

> $ asdf plugin-add python

```
updating plugin repository...HEAD is now at 00af419 feat: adding plugin for avalanchego (#1058)
spinlock@Derico:~/src/dotfiles$ asdf list-all python
2.1.3
2.2.3
2.3.7
2.4.0
2.4.1
2.4.2
2.4.3
snip
3.12.0
3.12-dev
3.12.1
3.12.2
3.12.3
3.12.4
3.12.5
3.12.6
3.12.7
3.13.0
3.13.0t
3.13-dev
3.13t-dev
3.14-dev
3.14t-dev
snip
stackless-3.4-dev
stackless-3.4.2
stackless-3.4.7
stackless-3.5.4
stackless-3.7.5
```

> $ asdf global python 3.12.7

```
version 3.12.7 is not installed for python
```

> $ asdf install python 3.12.7

```
python-build 3.12.7 /home/spinlock/.asdf/installs/python/3.12.7
Downloading Python-3.12.7.tar.xz...
-> https://www.python.org/ftp/python/3.12.7/Python-3.12.7.tar.xz
Installing Python-3.12.7...
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/spinlock/.asdf/installs/python/3.12.7/lib/python3.12/bz2.py", line 17, in <module>
    from _bz2 import BZ2Compressor, BZ2Decompressor
ModuleNotFoundError: No module named '_bz2'
WARNING: The Python bz2 extension was not compiled. Missing the bzip2 lib?
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/spinlock/.asdf/installs/python/3.12.7/lib/python3.12/ctypes/__init__.py", line 8, in <module>
    from _ctypes import Union, Structure, Array
ModuleNotFoundError: No module named '_ctypes'
WARNING: The Python ctypes extension was not compiled. Missing the libffi lib?
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'readline'
WARNING: The Python readline extension was not compiled. Missing the GNU readline lib?
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/spinlock/.asdf/installs/python/3.12.7/lib/python3.12/sqlite3/__init__.py", line 57, in <module>
    from sqlite3.dbapi2 import *
  File "/home/spinlock/.asdf/installs/python/3.12.7/lib/python3.12/sqlite3/dbapi2.py", line 27, in <module>
    from _sqlite3 import *
ModuleNotFoundError: No module named '_sqlite3'
WARNING: The Python sqlite3 extension was not compiled. Missing the SQLite3 lib?
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/spinlock/.asdf/installs/python/3.12.7/lib/python3.12/tkinter/__init__.py", line 38, in <module>
    import _tkinter # If this fails your Python may not be configured for Tk
    ^^^^^^^^^^^^^^^
ModuleNotFoundError: No module named '_tkinter'
WARNING: The Python tkinter extension was not compiled and GUI subsystem has been detected. Missing the Tk toolkit?
Installed Python-3.12.7 to /home/spinlock/.asdf/installs/python/3.12.7
```

> $ asdf global python 3.12.7

## Install Gigalixir

> pip3 install gigalixir --user

```
Collecting gigalixir
  Downloading gigalixir-1.13.1-py3-none-any.whl.metadata (835 bytes)
    snip
Downloading cryptography-43.0.1-cp39-abi3-manylinux_2_28_x86_64.whl (4.0 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 4.0/4.0 MB 347.1 kB/s eta 0:00:00
   snip
Downloading pycparser-2.22-py3-none-any.whl (117 kB)
Installing collected packages: zipp, urllib3, typing-extensions, six, setuptools, qrcode, pygments, pycparser, idna, click, charset-normalizer, certifi, requests, importlib-metadata, cffi, stripe, rollbar, cryptography, pyOpenSSL, gigalixir
Successfully installed certifi-2024.8.30 cffi-1.17.1 charset-normalizer-3.4.0 click-8.1.7 cryptography-43.0.1 gigalixir-1.13.1 idna-3.10 importlib-metadata-8.5.0 pyOpenSSL-24.2.1 pycparser-2.22 pygments-2.18.0 qrcode-8.0 requests-2.32.3 rollbar-1.0.0 setuptools-75.1.0 six-1.16.0 stripe-11.1.0 typing-extensions-4.12.2 urllib3-2.2.3 zipp-3.20.2
```

> bash/.bash_profile

```
PATH=$HOME/.local/bin:$PATH # Gigalixir tools
export PATH=$HOME/bin:./bin:$PATH
```

> $ gigalixir login:google

```
To login browse to https://api.gigalixir.com/oauth/google?session=snip
Would you like us to save your api key to your ~/.netrc file? [Y/n]:
Logged in as andrew.d.dixon@gmail.com.
```

> $ gigalixir account

```
{
  "api_key": "snip",
  "credit_cents": 0,
  "email": "andrew.d.dixon@gmail.com",
  "tier": "FREE"
}
```

# Create Gigalixir Application

> $ gigalixir create

```
Created app: wrathful-these-lobster.
Set git remote: gigalixir.
wrathful-these-lobster
```

> $ gigalixir login:google

```
To login browse to https://api.gigalixir.com/oauth/google?session=snip
Would you like us to save your api key to your ~/.netrc file? [Y/n]: 
Logged in as andrew.d.dixon@gmail.com.
spinlock@Derico:~/src/blog$ gigalixir account
{
  "api_key": "snip",
  "credit_cents": 0,
  "email": "andrew.d.dixon@gmail.com",
  "tier": "FREE"
}

```

# Create the Database

> $ gigalixir pg:create --free

```
A word of caution: Free tier databases are not suitable for production and migrating from a free db to a standard db is not trivial. Do you wish to continue? [y/N]: y
{
  "app_name": "wrathful-these-lobster",
  "database": "snip",
  "host": "postgres-free-tier-v2020.gigalixir.com",
  "id": "snip",
  "password": "pw-snip",
  "port": 5432,
  "state": "AVAILABLE",
  "tier": "FREE",
  "url": "postgresql://snip-user:pw-snip@postgres-free-tier-v2020.gigalixir.com:5432/snip",
  "username": "snip-user"
}
```

> $ gigalixir pg

```
[
  {
    "app_name": "wrathful-these-lobster",
    "database": "snip",
    "high_availability": "disabled",
    "host": "postgres-free-tier-v2020.gigalixir.com",
    "id": "snip",
    "limited_at": null,
    "password": "pw-snip",
    "port": 5432,
    "read_replicas": 0,
    "state": "AVAILABLE",
    "tier": "FREE",
    "url": "postgresql://snip-user:pw-snip@postgres-free-tier-v2020.gigalixir.com:5432/snip",
    "username": "snip-user"
  }
]
```

> $ gigalixir config

```
{
  "DATABASE_URL": "ecto://snip-user:pw-snip@postgres-free-tier-v2020.gigalixir.com:5432/snip",
  "POOL_SIZE": "2"
}
```

> $ gigalixir open

```
Running: xdg-open https://wrathful-these-lobster.gigalixirapp.com
Opening in existing browser session.
```

# Add SSH Keys

> $ gigalixir account:ssh_keys:add "$(cat ~/.ssh/id_rsa.pub)"

```
Please allow a few minutes for the SSH key to propagate to your run containers.
```

# Migrate the Database

> $ gigalixir ps:migrate

```
The authenticity of host '[v2018-us-central1.gcp.ssh.gigalixir.com]:30679 ([35.224.115.157]:30679)' can't be established.
ED25519 key fingerprint is SHA256:snip.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[v2018-us-central1.gcp.ssh.gigalixir.com]:30679' (ED25519) to the list of known hosts.

21:42:06.687 [info] == Running 20241010051754 Blog.Repo.Migrations.CreateUsers.change/0 forward

21:42:06.687 [info] create table users

21:42:06.705 [info] == Migrated 20241010051754 in 0.0s
Connection to v2018-us-central1.gcp.ssh.gigalixir.com closed.
```

## Debug static assets not serving in Gigalixir

> $ PHX_SERVER=true PORT=4000 _build/prod/rel/$APP_NAME/bin/$APP_NAME start

```
15:57:54.678 [info] Running BlogWeb.Endpoint with Bandit 1.5.7 at :::4000 (http)
15:57:54.686 [info] Access BlogWeb.Endpoint at https://example.com
15:58:14.559 request_id=F_05iXeUNSdRBUkAAAAD [info] GET /
15:58:14.561 request_id=F_05iXeUNSdRBUkAAAAD [info] Sent 200 in 2ms
15:58:19.667 request_id=F_05iqgHVJtRBUkAAAAj [info] GET /users
15:58:19.674 request_id=F_05iqgHVJtRBUkAAAAj [info] Sent 200 in 7ms
15:58:19.774 request_id=F_05iq5mQ7hRBUkAAABj [info] GET /assets/app.css
15:58:19.774 request_id=F_05iq5mQ7hRBUkAAABj [info] Sent 404 in 263µs
15:58:19.801 request_id=F_05irAGv9RRBUkAAACD [info] GET /assets/app.js
15:58:19.801 request_id=F_05irAGv9RRBUkAAACD [info] Sent 404 in 201µs
```

## Up Next ... Managing Node in ASDF
