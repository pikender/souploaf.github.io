---
layout: post
title: Elixir Phoenix Setup on Fresh Ubuntu Install
summary: Shared commands from Ubuntu Terminal History :)
categories: elixir
---

## [Install Elixir](http://elixir-lang.org/install.html#unix-and-unix-like)
    1  wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
    1' sudo dpkg -i erlang-solutions_1.0_all.deb
    2  sudo apt-get update
    3  sudo apt-get install elixir

## [Install Phoenix](http://www.phoenixframework.org/docs/installation)
    5  mix local.hex
    6  mix archive.install https://github.com/phoenixframework/phoenix/releases/download/v1.0.3/phoenix_new-1.0.3.ez

### Linux-only filesystem watcher that Phoenix uses for live code reloading
    7  sudo apt-get install inotify-tools

### Install Postgres
    9  sudo apt-get install postgresql

#### Install Postgres UI Admin Tool
    10  sudo apt-get install pgadmin3
    11  pgadmin3

### Install Nodejs
    15  sudo apt-get install nodejs

### Install npm
    28  sudo apt-get install npm
    29  npm install
    37  which node
    38  which nodejs
    39  ln -s /usr/bin/nodejs /usr/bin/node
    40  sudo ln -s /usr/bin/nodejs /usr/bin/node
    41  npm install

## Local Phoenix Development
    88  git clone https://github.com/pikender/phoenix
    89  cd phoenix/
    90  git remote add upstream https://github.com/phoenixframework/phoenix
    91  git config --list
    95  mix test
    96  mix deps.get
    97  mix test
    98  MIX_ENV=docs mix docs

## Local Elixir Development
    107  which elixir
    121  git clone https://github.com/pikender/elixir
    122  cd elixir/
    123  git remote add upstream https://github.com/elixir-lang/elixir
    125  make test
    126  sudo apt-get install erlang-parsetools erlang-dev  erlang-edoc erlang-eunit erlang-tools  erlang-common-test
    127  make test
    128  sudo apt-get install erlang-dialyzer
    129  make clean test

## Extras

### [Ubuntu System Reource Indicator](http://www.noobslab.com/2012/02/install-indicator-multiload-in-ubuntu.html)
    114  sudo add-apt-repository ppa:indicator-multiload/stable-daily
    115  sudo apt-get update
    116  sudo apt-get install indicator-multiload

### Install Vim
    24  sudo apt-get install vim
    58  sudo apt-get install ruby
    60  ruby -v
    61  sudo apt-get install rake
    63  sudo apt-get install vim-gnome
    65  sudo apt-get install curl
    66  curl -L https://bit.ly/janus-bootstrap | bash

### Install [An interactive shell for git](https://github.com/thoughtbot/gitsh)
    93  sudo apt-get install git-sh
    94  git-sh

## References

- [Elixir - Make clean test fails](https://github.com/elixir-lang/elixir/issues/3035)
- [Can not install packages using node package manager in Ubuntu](http://stackoverflow.com/questions/21168141/can-not-install-packages-using-node-package-manager-in-ubuntu)
