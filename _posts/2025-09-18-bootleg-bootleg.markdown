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
     {:ssh_client_key_api, github: "axelson/ssh_client_key_api", branch: "suppor
     {:credo, "~> 1.7", only: [:dev, :test], runtime: false},
     <snip>
   ]
 end
 ```

 We've also updated credo to the latest version.
