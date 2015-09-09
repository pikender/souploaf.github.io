---
layout: post
title: "Rails4 Upgrade: Devise Upgrade"
---

Rails 3 was using Devise 2.x.x and Rails 4 needed 3.x.x .

We did the change as needed and started getting weird issue of 

```ruby
TypeError: [125] is not a symbol
```

**Backtrace**

```ruby
/gems/activesupport-3.2.16/lib/active_support/inflector/methods.rb:230 in const_defined?
/gems/activesupport-3.2.16/lib/active_support/inflector/methods.rb:230 in block in constantize
/gems/activesupport-3.2.16/lib/active_support/inflector/methods.rb:229 in each
/gems/activesupport-3.2.16/lib/active_support/inflector/methods.rb:229 in constantize
/gems/devise-2.2.3/lib/devise/rails/warden_compat.rb:27 in deserialize
/gems/warden-1.2.3/lib/warden/session_serializer.rb:34 in fetch
```

On further inspection, we found that the key `warden.user.user.key`

*IS*

```ruby
"warden.user.user.key": "[[\"125\"], \"qfdfiyyfLsPGruC7oNfX\"]"
```
*WAS*

```ruby
"warden.user.user.key": "[\"User"\, [\"125\"], \"qfdfiyyfLsPGruC7oNfX\"]"
```

is an array which had User as first element and now is an integer i.e. user_id

Only way to resolve we found was to logout all users which is `piece of cake`, **Thanks to Rails**

- Generate a new secret key
  - `bundle exec rake secret`
- Change the secret\_token in `config/initializers/secret_token.rb` 
  - `Rails::Application.config.secret_token = 'TOKEN'`

**Enjoy Rails 4 !!!**
