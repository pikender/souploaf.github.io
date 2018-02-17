---
layout: post
title: Multi Tenant Subdomain *Local* Setup Using Ergo
---

# Day to Day Hustle

- Working on a [multi-tenant app](https://en.wikipedia.org/wiki/Multitenancy) and managing tenant access using sub-domain
- Ensuring all links in mails, views and any type of communication speaks of multi-tenant and no the host
- Ensuring that most of the expectations are met and tested while developing application locally
- Staging deploys uncover infra-related issues than code-related
- Production deploys are seamless as code and infra related issues are already sorted

# Application Stack and Resolutions

- [Ruby on Rails](http://rubyonrails.org/), will refer to as **RoR** further, is our choice of web framework ad it supports multitenancy requirements quite elegantly
- There are configurations already in the framework to prepare links as you desire
  - Challenge there is to make an application level configuration ( mailer host ) work as tenant level configuration
  - like mail communication, other form of communication also has to adapt as per tenant
- All the above is possible one or other way but testing all this in **development environment** is still not that pleasant and has lot of gotchas & painful configurations

## Almost universally known approach

- Modify hosts file `/etc/hosts` to mimick all the subdomains under test
- Now, whenever you type `subdomain.host` ( e.g. pikender.vibrantstack.com ), it knows where to route this request
- RoR in development runs on a `3000` port so urls will be of form `pikender.vibrantstack.com:3000` and in staging/production environment as RoR servers are reached through [nginx](https://www.nginx.com/) or [Apache](https://httpd.apache.org/) which caters to `pikender.vibrantstack.com` as default port to use is `80/443` based on `http/https`
  - So, a part of production setup (host) is mimicked but not as whole (host:port)
- Urls as sent in mailers or at other places need to be injected with port info to make it work and test the behavior
- Manual change of urls to add port info keeps it unpleasant and can lead to leaking bugs and assumptions which can bite us hard in production ( nobody wants unhappy or agitated customers )

## Mimicking the port in url in local dev environment

- I take great pleasure in introducing [ergo](https://github.com/cristianoliveira/ergo) which solves the very problem in hand

> **The management of multiple local services running on different ports made easy**

### Motivation

> Dealing with multiple apps locally, and having to remember each port representing each microservice is frustrating. I wanted a simple way to assign each service a proper local domain. Ergo solves this problem.

# [Ergo](https://github.com/cristianoliveira/ergo) easing up developer's Day-to-Day Hustle

## Linux Setup

- Create an ergo file

  ```
  # ~/.ergo
  # Note the no use of top-level domain like .com or .net etc
  # that's where the fun is :)

  app.vibrantstack http://localhost:3000
  blog.vibrantstack http://localhost:3000
  analytics.vibrantstack http://localhost:3000
  ```

- Run ergo [refer options here](https://github.com/cristianoliveira/ergo#ergos-configuration)

  ```sh
  # Run on terminal
  # Now app.vibrantstack.mdn should be accessible
  # Note we defined app.vibrantstack in ~/.ergo and
  # .mdn is the tld we want ergo to serve from configuration
  # so app.vibrantstack.mdn will be served by localhost:3000

  ergo run -domain .mdn
  ```

- Run Google Chrome with proxy
  - We need to make Google Chrome aware of our new shiny proxy :)

  ```sh
  # Google Chrome launched with proxy respect app.vibrantstack.mdn
  # and ergo will serve from localhost:3000
  # As end-user you will get a feel of accessing real url with .mdn tld :)

  google-chrome --proxy-pac-url=http://localhost:2000/proxy.pac
  ```

- Making Curl commands

  ```sh
  # Like Google Chrome, we need to make termial also aware of our proxy
  # Voila !!!, now curl commands work too, Hurray !!

  export http_proxy=http://127.0.0.1:2000/
  ```

- [httparty](https://github.com/jnunemaker/httparty) library used for making http api calls

  ```rb
  # Sample httparty class making http api requests
  module AnalyticsApi
    class Request
      include HTTParty
    end
  end
  ```

  ```rb
  # config/environments/development.rb
  Rails.application.configure do
    ## Other configurations

    ## httparty also can be made aware of proxy like below
    config.to_prepare do
      AnalyticsApi::Request.class_eval do
        http_proxy '127.0.0.1', 2000, 'user', 'pass'
      end
    end
  end
  ```

# Developer's piece of mind

- Emails can be tested with correct links and be assured that emails sent from production would also have right links
  - with right setup, links in emails would be like `app.vibrantstack.mdn` for production counterpart of `app.vibrantstack.com`
- Internal micro-services like `analytics` can be mimicked to use local service by setting proxy in [httparty](https://github.com/jnunemaker/httparty)
- Instead of accessing local app using `http://localhost:3000`, you can use more lively urls like `http://app.vibrantstack.mdn` or `http://blog.vibrantstack.mdn`
- Rounding back to multitenancy, tenants dashboard's can be accessed **in local development** using `http://firstclient.vibrantstack.mdn` and another could be `http://awesomeclient.vibrantstack.mdn`

## Business Advantages

- Most of the privacy related issues / embarassing moments can be avoided as you get to mimick real production setup locally, if its achieved 100% your business is on a right trajectory

**Happy to help dev's have a piece of mind to take on the next horrendous task ;)**

