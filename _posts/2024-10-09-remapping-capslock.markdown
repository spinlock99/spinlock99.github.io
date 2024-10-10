---
layout: post
title:  "Remapping CAPSLOCK in Ubuntu"
date:   2024-10-09 10:45:00 -0700
categories: os
---
{% include google_analytics.html %}

## Upgrade
I upgraded Ubuntu to 24.10 and my keybindings stopped working. I like to have
CAPSLOCK set as a hyper key that acts like ESC when tapped and CTRL when used
as a modifier. I then make the ESC key act as CAPSLOCK.

After trying various options including:
* gnome-tweeks
* other stuff that I forget because the blog wasn't working yet so I wasn't
writing stuff down.

I decided to use `keyd` because it works directly with the kernel so it's more
agnostic than other options which work with `X` or `Wayland`.

## Keyd

> Source

[Github Repo](https://salsa.debian.org/rhansen/keyd)

> Install

[Ubuntu Packages](https://launchpad.net/~keyd-team/+archive/ubuntu/ppa)

```
$ sudo add-apt-repository ppa:keyd-team/ppa
$ sudo apt update
$ sudo apt install keyd
```

> Configure

`/etc/keyd/default.conf`
```
[ids]

*

[main]

# Maps capslock to escape when pressed and control when held.
capslock = overload(control, esc)

# Remaps the escape key to capslock
esc = capslock
```

configure in dotfiles?

## DotFiles

## Document It
I checked out this Jekyll blog from github and made a few changes. But, when I
pushed `master` to `github`, I found out that they'd changed their build system
and I'd need to fix some `ruby` dependencies:

```
Annotations
1 warning
build
The github-pages gem can't satisfy your Gemfile's dependencies. If you want to use a different Jekyll version or need additional dependencies, consider building Jekyll site with GitHub Actions: https://jekyllrb.com/docs/continuous-integration/github-actions/
```

So, I have to install `ruby` locally...

## ASDF

I'm alread trying to use asdf for elixir so I might as well use it for Ruby too.

### Download the Source

`$git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.14.1`

### Edit `.bashrc`

```
. "$HOME/.asdf/asdf.sh"
. "$HOME/.asdf/completions/asdf.bash"
```

### Install Ruby

`$ asdf plugin add ruby`

[followed blog here](https://mac.install.guide/rubyonrails/7)

[no version set](https://github.com/asdf-vm/asdf/issues/557)

## Putting it All Together

```
$ gem install jekyll bundler
$ bundle exec jekyll serve
```
