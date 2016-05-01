---
layout: post
title: "Rails4 Upgrade: XML Params Not Parsed"
---

After Rails 4 Upgrade, one of our 3rd Party Integration *Quickteller*
started failing.

No Transactions found post Rails4 Deploy when Reports Generated Later
day

On inspecting further, we observed that Callback is failing at same
point with message `NoMethodError - undefined method '[]' for nil:NilClass:` 
and had no `Parmeters:`

**_Twist in the tale is that service sends xml-encoded parameters and xml
parameter parsing is removed from Rails 4 Core and needs to be included
back to use the functionality :P_**


[A XML parameters parser for Action Pack (removed from core in Rails 4.0)](https://github.com/rails/actionpack-xml_parser)

- Include this gem into your Gemfile:
  - `gem 'actionpack-xml_parser'`
- Then, add ActionDispatch::XmlParamsParser middleware after ActionDispatch::ParamsParser in config/application.rb:
  - `config.middleware.insert_after ActionDispatch::ParamsParser, ActionDispatch::XmlParamsParser`
- You may need to require the ActionDispatch::XmlParamsParser manually. Add the following in your config/application.rb:
  - `require 'action_dispatch/xml_params_parser'`
  

**Enjoy Rails 4 !!!**
