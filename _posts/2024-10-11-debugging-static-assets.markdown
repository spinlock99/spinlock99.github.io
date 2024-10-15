---
layout: post
title:  "Debugging Static Assets"
date:   2024-10-11 07:00:00 -0700
categories: elixir phoenix gigalixir
---
{% include google_analytics.html %}

## Busted Assets

# 404s in the Browser

![404s in the Browser](/assets/2024-10-11-404s-in-the-browser.png)

# 404s in the Logs

_scroll right_

```
2024-10-11T17:24:29.370751+00:00 wrathful-these-lobster[b'wrathful-these-lobster-6f5d75f849-6g9wb']: web.1  | 17:24:29.369 request_id=3580b827db98cdacd46adcd5b88a58fb [info] GET /
2024-10-11T17:24:29.370773+00:00 wrathful-these-lobster[b'wrathful-these-lobster-6f5d75f849-6g9wb']: web.1  | 17:24:29.369 request_id=3580b827db98cdacd46adcd5b88a58fb [info] Sent 200 in 280µs
2024-10-11T17:24:38.092536+00:00 wrathful-these-lobster[b'wrathful-these-lobster-6f5d75f849-6g9wb']: web.1  | 17:24:38.091 request_id=43a553ee2d39459995cf65274217e94b [info] GET /
2024-10-11T17:24:38.092560+00:00 wrathful-these-lobster[b'wrathful-these-lobster-6f5d75f849-6g9wb']: web.1  | 17:24:38.091 request_id=43a553ee2d39459995cf65274217e94b [info] Sent 200 in 234µs
2024-10-11T17:24:44.991412+00:00 wrathful-these-lobster[b'wrathful-these-lobster-6f5d75f849-6g9wb']: web.1  | 17:24:44.990 request_id=63e507084c4860554724962dcfafd2e9 [info] GET /assets/app.css
2024-10-11T17:24:44.991437+00:00 wrathful-these-lobster[b'wrathful-these-lobster-6f5d75f849-6g9wb']: web.1  | 17:24:44.990 request_id=63e507084c4860554724962dcfafd2e9 [info] Sent 404 in 283µs
```

# Build Packs

> .buildpacks

```
https://github.com/gigalixir/gigalixir-buildpack-elixir
https://github.com/gigalixir/gigalixir-buildpack-phoenix-static
https://github.com/gigalixir/gigalixir-buildpack-releases.git
```

[Gigalixir Buildpack Phoenix Static](https://github.com/gigalixir/gigalixir-buildpack-phoenix-static)

> phoenix_static_buildpack.config

```
# Clean out cache contents from previous deploys
clean_cache=false

# We can change the filename for the compile script with this option
compile="compile"

# We can set the version of Node to use for the app here
node_version=12.13.1

# We can set the version of NPM to use for the app here (defaults to "latest")
npm_version=8.19.2

# We can set the version of Yarn to use for the app here
#yarn_version=1.13.0

# We can set the path to phoenix app. E.g. apps/phoenix_app when in umbrella.
phoenix_relative_path=.

# Remove node and node_modules directory to keep slug size down if it is not needed.
remove_node=false

# We can change path that npm dependencies are in relation to phoenix app. E.g. assets for phoenix 1.3 support.
assets_path=assets/

# We can change phoenix mix namespace tasks. E.g. `phoenix` for phoenix < 1.3 support.
phoenix_ex=phx
```

> compile

```
# app_root/compile
cd $phoenix_dir
npm --prefix ./assets run deploy
mix "${phoenix_ex}.digest" #use the ${phoenix_ex} variable instead of hardcoding phx or phoenix
```

# Recreating the Bug Locally
