---
layout: post
title:  "Running Phoenix with Systemd"
date:   2024-10-17 07:00:00 -0700
categories: elixir phoenix gigalixir
---
{% include google_analytics.html %}

## Systemd: a system and service manager for Linux

When run as first process on boot (as PID 1), it acts as init system that
brings up and maintains userspace services. Separate instances are started for
logged-in users to start their services.

# Concepts

**systemd** provides a dependency system between various entities called **units**
of 11 different **types**. **Units** encapsulate various objects that are relevant
for system **boot-up** and **maintenance**. The various **unit types** may have a
number of additional substates, which are mapped to the five generalized unit
states. We will be managing our Phoenix App -- Haikuter -- as a **service unit**.
**Service units** start and control daemons and the processes they consist of.
For details, see systemd.service(5).

> We will be using the root user to configure and run the haikuter service.

Capistrano ran the remote commands on the server as builder. We are working
directly on the staging server at this point as root.

[The Phoenix Release Docs](https://hexdocs.pm/phoenix/releases.html)
tell us to set the environment variables first. We'll define these in the
systemd service definition file so that the haikuter process will have access to them
when it starts up. I tried putting the environment variables in `/etc/environment`
first but it looks like systemd runs without access to those variables. However,
it does feel cleaner to isolate data that should only be used by haikuter into
the service file that defines how it will be run. This allows other users on the
system to not have their own environments clobbered.

**SECRET_KEY_BASE** is generated with mix.

> $ mix phx.gen.secret

**DATABASE_URL** holds the credentials for Postgres.

**MIX_ENV** tells us we're in production.

**PORT** keep it in user space for now. This will eventually be 8080 when ssl
is configured.

**PHX_SERVER** tells Phoenix to run the web server on the given host.

**PHX_HOST** tells Phoenix what its hostname is.

> spinlock@Derico:~/src/haikuter$ sudo vi /etc/systemd/system/haikuter.service

```
[Unit]
Description=Haikuter Daemon

[Service]
Type=simple
User=builder
Group=builder
Restart=on-failure

Environment=PORT=4001
Environment=MIX_ENV=prod
Environment=PHX_SERVER=true
Environment=PHX_HOST=staging.haikuter.com
Environment=DATABASE_URL=ecto://username:password@staging.haikuter.com/haikuter_prod
Environment=SECRET_KEY_BASE=snip
Environment=LANG=en_US.UTF-8

WorkingDirectory=/var/www/haikuter/current/_build/prod/rel/haikuter/
ExecStart=/var/www/haikuter/current/_build/prod/rel/haikuter/bin/haikuter start
ExecStop=/var/www/haikuter/current/_build/prod/rel/haikuter/bin/haikuter stop

[Install]
WantedBy=multi-user.target
```

> spinlock@Derico:~/src/haikuter$ sudo systemctl start haikuter

```
Warning: The unit file, source configuration file or drop-ins of haikuter.service changed on disk. Run 'systemctl daemon-reload' to reload units.
```

> spinlock@Derico:~/src/haikuter$ sudo systemctl daemon-reload

> spinlock@Derico:~/src/haikuter$ sudo systemctl start haikuter

> spinlock@Derico:~/src/haikuter$ tail -f /var/log/syslog

```
systemd[1]: Started haikuter.service - Haikuter Daemon.
haikuter[293272]: 08:52:07.218 [info] Running HaikuterWeb.Endpoint with Bandit 1.5.7 at :::4001 (http)
haikuter[293272]: 08:52:07.224 [info] Access HaikuterWeb.Endpoint at https://staging.haikuter.com
google-chrome.desktop[293364]: Opening in existing browser session.
haikuter[293272]: 08:52:45.211 request_id=F_9IYWz72PNnP4YAAAjE [info] GET /
haikuter[293272]: 08:52:45.216 request_id=F_9IYWz72PNnP4YAAAjE [info] Sent 200 in 5ms
```

# Restarting the Service on Deploy

Our `capistrano` deploy script will run as the `builder` user which does not have
permission to start and stop daemons using `systemctl`. So, we need to create
a SystemD service to watch `/var/www/haikuter/current` and restart `haikuter`
when it changes:

> spinlock@Derico:~/src/haikuter$ cat /etc/systemd/system/haikuter-watcher.service 

```
[Unit]
Description=Haikuter Restarter
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/systemctl restart haikuter.service

[Install]
WantedBy=multi-user.target
```

> spinlock@Derico:~/src/haikuter$ cat /etc/systemd/system/haikuter-watcher.path 

```
[Path]
PathModified=/var/www/haikuter/current/

[Install]
WantedBy=multi-user.target
```

One problem with `haikuter-watcher-path` is that it will not recognize the symlink
to `/var/www/haikuter/current` being changed by `capistrano` during the deploy.
Instead, we touch `/var/www/haikuter/current` after the directory is moved:

```
namespace :symlink do
  task :release do
    invoke "deploy:symlink:restart_service"
  end

  task :restart_service do
    on roles(:app) do |host|
      execute(:touch, '/var/www/haikuter/current')
    end
  end
end
```
