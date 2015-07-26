---
layout: post
title: cannot load such file -- mysql2/mysql2 (LoadError) 
---

While upgrading to Rails 4, lot of gem dependencies change as expected

To keep the dependencies clean, we decided to create a new gemset in rvm
to install everything fresh and ensuring no existing dependencies / gems
preventing us from any surprise on SERVER deploy

What you get different in a new gemset is that all other gems installed
long time back and working are installed again and all the linking /
dependency resolution is done again which might throw a surprise.

### Why new rvm gemset

It might not be relevant / seen futile by developers already working on
project as everything is installed and working for them but is highly
critical for NEW developers who would join or to install on new machines
too for existing developers

### Everything starts with de-facto

```ruby
bundle install
```

### And here come's the surprise
mysql2 gem failed to install

After a little bit of [search](https://github.com/brianmario/mysql2/issues/192#issuecomment-8701366) came across below solution and mysql2 gem
install worked

```ruby
gem install mysql2 -v '0.3.13'  -- --srcdir=/usr/local/mysql/include
```

Happy and resumed `bundle install`. It finished smoothly.

Fired the Rails Server `bundle exec rails s` 

### Boom !!!

Below Error shown in backtrace

```ruby
/usr/local/share/gems/gems/mysql2-0.3.13/lib/mysql2.rb:8:in `require': cannot load such file -- mysql2/mysql2 (LoadError)
```

Browsing through stackoverflow and mention of above errors, landed on [mysql2 issue](https://github.com/brianmario/mysql2/issues/192#issuecomment-22910390)
and [couldn't understand kern.osversion `14.0.0'](http://stackoverflow.com/questions/28068383/why-does-homebrew-report-couldnt-understand-kern-osversion-14-0-0)

With the help of pointers in above references, fixed it by editing
`Makefile` to use `CC` as `/usr/bin/gcc` instead of
`/usr/local/bin/gcc-4.2`

Its always better to install new projects under different `rvm gemsets`
to better know / handle adaptation to changes in OS due to upgrades or
otherwise.

References
==========

- https://github.com/brianmario/mysql2/issues/192#issuecomment-22910390
- http://stackoverflow.com/questions/28068383/why-does-homebrew-report-couldnt-understand-kern-osversion-14-0-0
