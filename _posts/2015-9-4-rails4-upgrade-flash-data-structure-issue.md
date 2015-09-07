---
layout: post
title: "Rails4 Upgrade: Rollback / Downgrade to Rails 3"
---

As with most upgrades / deploy on production, if things go awry, we may have to rollback.

Due to some unexpected behavior, we decided to rollback Rails 4 Upgrade and Boom !! started getting lot of exceptions

```ruby
NoMethodError: undefined method 'sweep' for Hash
```

as session cookies blowed up due to different Data Structures in Rails 4 & 3 and not backward-compatible :(

- Created config/initializers/rails4_to_rails3_downgradability.rb
  - Refer <a href="https://gist.github.com/henrik/bb6732d5d4cddb5085a4" target="_blank">gist</a>

```ruby
# Without this fix, downgrading from Rails 4 to Rails 3 causes session cookies to blow up.
#
# The way the flash is stored in the session changed in a backwards-incompatible way.

if Rails::VERSION::MAJOR == 3

  module ActionDispatch
    class Flash
      def call(env)
        if (session = env['rack.session']) && (flash = session['flash'])

          # Beginning of change!

          if flash.respond_to?(:sweep)
            flash.sweep
          else
            session.delete("flash")
          end

          # End of change!

        end

        @app.call(env)
      ensure
        session    = env['rack.session'] || {}
        flash_hash = env[KEY]

        if flash_hash
          if !flash_hash.empty? || session.key?('flash')
            session["flash"] = flash_hash
            new_hash = flash_hash.dup
          else
            new_hash = flash_hash
          end

          env[KEY] = new_hash
        end

        if session.key?('flash') && session['flash'].empty?
          session.delete('flash')
        end
      end
    end
  end
end
```

**Enjoy Rails 4 !!!**
