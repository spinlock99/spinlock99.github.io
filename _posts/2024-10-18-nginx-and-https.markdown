---
layout: post
title:  "Nginx and HTTPS"
date:   2024-10-18 07:00:00 -0700
categories: elixir phoenix
---
{% include google_analytics.html %}

## Nginx: A High Performance Web Server

Nginx is a webserver or reverse web proxy depending on who you ask and what
documents they are reading from. I'm not sure what the difference is but I do
know that Nginx works verry well for terminating and HTTPS connection and
passing requests down to the Phoenix app.

# Running over http

> spinlock@Derico:~/src/haikuter$ cat nginx/haikuter.conf

```
upstream phoenix {
	server 127.0.0.1:4001 max_fails=5 fail_timeout=60s;
}

server {
  server_name staging.haikuter.com;

  location / {
    proxy_http_version 1.1;

    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header X-Cluster-Client-Ip $remote_addr;
    proxy_set_header X-Forward-Host $host;
    proxy_set_header X-Forward-Port $server_port;
    proxy_set_header X-Forward-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;

    proxy_pass http://phoenix;
  }

  listen 80;

#    listen 443 ssl; # managed by Certbot
#    ssl_certificate /etc/letsencrypt/live/app.barronwatchcompany.com/fullchain.pem; # managed by Certbot
#    ssl_certificate_key /etc/letsencrypt/live/app.barronwatchcompany.com/privkey.pem; # managed by Certbot
#    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
#    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}


#server {
#    if ($host = app.barronwatchcompany.com) {
#        return 301 https://$host$request_uri;
#    } # managed by Certbot
#
#
#        server_name app.barronwatchcompany.com;
#
#        listen 80;
#        listen [::]:80;
#    return 404; # managed by Certbot
#
#
#}
```

# Now HTTPS

https://letsencrypt.org/docs/certificates-for-localhost/

> spinlock@Derico:~$ openssl req -x509 -out localhost.crt -keyout localhost.key   -newkey rsa:2048 -nodes -sha256   -subj '/CN=localhost' -extensions EXT -config <(    printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")

```
.+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...........+..+...+...+.+...............+........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.........+......+...+...+....+.........+..+....+.........+......+.....+..........+.....+.............+..+.+........+.+...+......+...........+.+..+..............................+.+..+...+.+......+..+......+.......+..+..........+...+..+......+.+...+......+.........+..........................+.......+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
..............+....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*......+...+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.....+......+.+...+...............+..+...+.........+.+...........+.+........+......+.+........+.+....................+...............+...+.+..+....+...+.....+.+.....+.+...............+...+............+.....+....+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
```

https://documentation.ubuntu.com/server/how-to/security/install-a-root-ca-certificate-in-the-trust-store/#install-a-root-ca-certificate-in-the-trust-store

> spinlock@Derico:~$ sudo cp localhost.crt /usr/local/share/ca-certificates/

> spinlock@Derico:~$ sudo update-ca-certificates

```
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
Processing triggers for ca-certificates-java (20240118) ...
Adding debian:localhost.pem
done.
done.
```

# Trying Again

Chrome doesn't like the self signed cert above and will complain. By adding a
certificate authority and using it to sign the staging certificate, we can add
the certificate authority to chome so ssl works as intended.

> spinlock@Derico:~/src/haikuter$ cat nginx/self-signed-certificate.sh

```
#!/usr/bin/env bash

CA_NAME=derico-ca
DOMAIN_NAME=haikuter.com # Use your own domain name

######################
# Become a Certificate Authority
######################

# Generate private key
openssl genrsa -des3 -out $CA_NAME.key 2048
# Generate root certificate
openssl req -x509 -new -nodes -key $CA_NAME.key -sha256 -days 825 -out $CA_NAME.pem

######################
# Create CA-signed certs
######################

# Generate a private key
openssl genrsa -out $DOMAIN_NAME.key 2048
# Create a certificate-signing request
openssl req -new -key $DOMAIN_NAME.key -out $DOMAIN_NAME.csr
# Create a config file for the extensions
>$DOMAIN_NAME.ext cat <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = $DOMAIN_NAME # Be sure to include the domain name here because Common Name is not so commonly honoured by itself
DNS.2 = staging.$DOMAIN_NAME # Optionally, add additional domains (I've added a subdomain for the staging server)
#IP.1 = 127.0.0.1 # Optionally, add an IP address (if the connection which you have planned requires it)
EOF
# Create the signed certificate
openssl x509 -req -in $DOMAIN_NAME.csr -CA $CA_NAME.pem -CAkey $CA_NAME.key -CAcreateserial \
-out $DOMAIN_NAME.crt -days 825 -sha256 -extfile $DOMAIN_NAME.ext
```

> $ sudo cp haikuter.com.crt /etc/ssl/certs/

> $ sudo cp haikuter.com.key /etc/ssl/private/

> $ sudo cp derico-ca.pem /usr/local/share/ca-certificates/

> $ sudo update-ca-certificates

> spinlock@Derico:~/src/haikuter$ cat nginx/haikuter.conf

Verify the new certificate:

> $ openssl verify -CAfile derico-ca.pem -verify_hostname staging.haikuter.com haikuter.com.crt

Add your certificate authority to Chrome:

settings -> Privacy and security -> Security -> Manage certificates -> Authoities -> Import
```
upstream phoenix {
	server 127.0.0.1:4001 max_fails=5 fail_timeout=60s;
}

server {
  server_name staging.haikuter.com;

  location / {
    proxy_http_version 1.1;

    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header X-Cluster-Client-Ip $remote_addr;
    proxy_set_header X-Forward-Host $host;
    proxy_set_header X-Forward-Port $server_port;
    proxy_set_header X-Forward-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;

    proxy_pass http://phoenix;
  }

  listen 443 ssl;
  ssl_certificate /etc/ssl/certs/haikuter.com.crt;
  ssl_certificate_key /etc/ssl/private/haikuter.com.key;
}


server {
  if ($host = staging.haikuter.com) {
      return 301 https://$host$request_uri;
  }


  server_name staging.haikuter.com;

  listen 80;
  listen [::]:80;
  return 404;
}
```

# Run the Migrations

> $ sudo systemctl restart nginx

> builder@Derico:/var/www/haikuter/current$ export DATABASE_URL=ecto://postgres:postgres@staging.haikuter.com/haikuter_prod

> builder@Derico:/var/www/haikuter/current$ echo $DATABASE_URL

```
ecto://postgres:postgres@staging.haikuter.com/haikuter_prod
```

> builder@Derico:/var/www/haikuter/current$ export SECRET_KEY_BASE=snip
> builder@Derico:/var/www/haikuter/current$ _build/prod/rel/haikuter/bin/haikuter eval "Haikuter.Release.migrate"

```
15:14:10.703 [info] == Running 20241017170942 Haikuter.Repo.Migrations.CreateUsers.change/0 forward
15:14:10.706 [info] create table users
15:14:10.747 [info] == Migrated 20241017170942 in 0.0s
```

> spinlock@Derico:~/src/haikuter$ sudo vi /etc/systemd/system/haikuter.service

```
[Unit]
Description=Haikuter Daemon

[Service]
Type=simple
User=builder
Group=builder
Restart=on-failure
Environment=LANG=en_US.UTF-8
Environment=MIX_ENV=prod
Environment=PHX_SERVER=true
Environment=PORT=4001
Environment=DATABASE_URL=ecto://postgres:postgres@staging.haikuter.com/haikuter_prod
Environment=SECRET_KEY_BASE=snip
Environment=PHX_HOST=staging.haikuter.com

WorkingDirectory=/var/www/haikuter/current/
ExecStartPre=/var/www/haikuter/current/_build/prod/rel/haikuter/bin/haikuter "Haikuter.Release.migrate"
ExecStart=/var/www/haikuter/current/_build/prod/rel/haikuter/bin/haikuter start
ExecStop=/var/www/haikuter/current/_build/prod/rel/haikuter/bin/haikuter stop

[Install]
WantedBy=multi-user.target
```
