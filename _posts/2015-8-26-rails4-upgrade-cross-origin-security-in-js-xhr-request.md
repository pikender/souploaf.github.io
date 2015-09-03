---
layout: post
title: "Rails4 Upgrade: Cross Origin Security in js/xhr Requests"
---

After Rails 4 Upgrade, we started getting `ActionController::InvalidCrossOriginRequest` Exception almost every minute

```ruby
ActionController::InvalidCrossOriginRequest at /method
======================================================

> Security warning: an embedded <script> tag on another site requested
protected JavaScript. If you know what you're doing, go ahead and
disable forgery protection on this action to permit cross-origin
JavaScript embedding.
```

Airbrake classified most errors under 2 actions, search and assets url

On asking Google about above error, instantly a new change [CSRF protection from cross-origin script tags](https://github.com/rails/rails/pull/13345) in Rails 4
for security popped.

Going through the details, source and intent of exception was clear.

As the error is quite on face, there are some measures for workaround
and fix.

*Solution 1 - We used*

[Override `non_xhr_javascript_response?` to bypass the
change](https://github.com/rails/rails/pull/13345#issuecomment-71099064)

```ruby
  def non_xhr_javascript_response?
    if request.get?
      super
    end
  end
```

*Solution 2*

[Reorder the respond_to formats to make html as default for requests
with format=\*/\*](https://github.com/rails/rails/pull/13345#issuecomment-71114660)

# Why the first solution ?

- To support mix of http / https on the site, assets url are relative so
  that browser can fill the protocol as per the page protocol
- _**bots**_ follow the different convention of prepending page absolute
  url to links starting with / and hence the error
  - //assets3.xyz.com/all.js becomes xyz.com//assets3.xyz.com/all.js
  - it hits the application for request and gets swallowed by
    application#render_not_found catch all errors
  - as .js *at-end* so interpreted as JS request and new security patch
    blocks it and boom the EXCEPTION :(
- For search action, it can be resolved with *Solution 2* but why to
  bother if we need to by-pass for assets anyways :P

#### Follow-up

- Relative Url Link Follow by bots might be resolved by configuration in
  robots.txt
  - More on this later ... cya :)

**Enjoy Rails 4 !!!**

*Reference*

- [PR](https://github.com/rails/rails/pull/13345)
- [commit](https://github.com/jeremy/rails/commit/1650bb3d56897cfef4c7e6b86a36eed4f1a41df5)

*Fix*

- [Fix 1](https://github.com/rails/rails/pull/13345#issuecomment-71099064)
- [Fix 2](https://github.com/rails/rails/pull/13345#issuecomment-71114660)

