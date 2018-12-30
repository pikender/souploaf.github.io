---
layout: post
title: Phoenix Vue Starter Setup
---

# Setup

## Install asdf to manage erlang, elixir, nodejs

- [asdf](https://github.com/asdf-vm/asdf)
- [asdf-erlang](https://github.com/asdf-vm/asdf-erlang)
- [asdf-elixir](https://github.com/asdf-vm/asdf-elixir)
- [asdf-nodejs](https://github.com/asdf-vm/asdf-nodejs)

### Commands which would be of help

- asdf plugin-add erlang https://github.com/asdf-vm/asdf-erlang.git
- asdf plugin-add elixir https://github.com/asdf-vm/asdf-elixir.git
- asdf plugin-add nodejs https://github.com/asdf-vm/asdf-nodejs.git
- asdf plugin-list
- asdf plugin-remove erlang
- asdf plugin-remove elixir
- asdf plugin-remove nodejs
- asdf list-all elixir
- asdf list-all nodejs
- asdf install elixir 1.6.6-otp-21
- asdf global elixir system
- asdf global nodejs system

### Create a .tool-versions file

```
elixir 1.7.4-otp-21
nodejs 10.15.0
```

- Run `asdf install` to install elixir and nodejs as mentioned in `.tool-versions` file

### Install Phoenix

- mix archive.install hex phx_new 1.4.0
- mix phx.new --version

## Create Phoenix Application

- mix phx.new activity_streams
- mix deps.get
- cd assets && npm install && node node_modules/webpack/bin/webpack.js --mode development
- mix ecto.create
- iex -S mix phx.server

## Vue Integration changes

- Check diff [Vue Integration](https://github.com/pikender/phx-vue-starter/pull/1/files)
