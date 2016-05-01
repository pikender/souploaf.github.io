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

  <script src="https://gist.github.com/pikender/faf1cefdca88f1580b98c0b66c9cb1cd.js"></script>

**Enjoy Rails 4 !!!**
