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

```ruby

Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

    /Users/pikender/.rvm/rubies/ruby-2.0.0-p451/bin/ruby extconf.rb 
checking for ruby/thread.h... yes
checking for rb_thread_call_without_gvl() in ruby/thread.h... yes
checking for rb_thread_blocking_region()... yes
checking for rb_wait_for_single_fd()... yes
checking for rb_hash_dup()... yes
checking for rb_intern3()... yes
-----
Using mysql_config at /usr/local/mysql/bin/mysql_config
-----
checking for mysql.h... yes
checking for errmsg.h... yes
checking for mysqld_error.h... yes
-----
Setting rpath to /usr/local/mysql/lib
-----
creating Makefile

make "DESTDIR=" clean

make "DESTDIR="
compiling client.c
compiling infile.c
compiling mysql2_ext.c
compiling result.c
linking shared-object mysql2/mysql2.bundle
couldn't understand kern.osversion `14.5.0'
ld: warning: directory not found for option
'-L/Users/travis/.sm/pkg/active/lib'
ld: -rpath can only be used when targeting Mac OS X 10.5 or later
collect2: ld returned 1 exit status
make: *** [mysql2.bundle] Error 1

make failed, exit code 2

Gem files will remain installed in
/Users/pikender/.rvm/gems/ruby-2.0.0-p451@easyerrands/gems/mysql2-0.3.20
for inspection.
Results logged to
/Users/pikender/.rvm/gems/ruby-2.0.0-p451@easyerrands/extensions/x86_64-darwin-12/2.0.0-static/mysql2-0.3.20/gem_make.out
An error occurred while installing mysql2 (0.3.20), and Bundler cannot
continue.
Make sure that `gem install mysql2 -v '0.3.20'` succeeds before
bundling.
```

# Solution 1

```sh
export MACOSX_DEPLOYMENT_TARGET=10.5
```

Then `bundle install` and application will run smoothly :)

[More Details](http://stackoverflow.com/a/26501617/306686)

# Solution 2

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
