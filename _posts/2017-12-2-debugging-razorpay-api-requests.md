---
layout: post
title: Debugging Razorpay ( Ruby ) Api Requests
---

[Razorpay](https://razorpay.com/) is one of the best payment gateway in India. With Subscriptions and other products, it surely has taken over other alternatives.

Razorpay APIs are available for use but SDK's are not yet updated fully to interact with them. While integrating subscriptions, we took a stride to update the SDK for our needs.

We were integrating with Ruby on Rails App and wanted to use [Razorpay Ruby SDK](https://github.com/razorpay/razorpay-ruby).

As we were dealing with Razorpay platform and ruby-sdk first time, we wanted to be sure of API calls being sent in correct way and wanted to analyse the received response in detail.

Razorpay Ruby SDK uses [httparty](https://github.com/jnunemaker/httparty) for making api calls which provides a way to enable request/response log using [debug_output](http://www.rubydoc.info/github/jnunemaker/httparty/HTTParty%2FClassMethods%3Adebug_output) set as `$stdout`. Refer the [blog](http://natritmeyer.com/blog/2013/09/09/how-to-debug-httparty-requests/) for the details and usage.

As we were already making changes to [razorpay-ruby](https://github.com/razorpay/razorpay-ruby) fork for subscription api wrappers, we made the change in `lib/razorpay/request.rb` directly to set `debug_output` and integrated needed api wrappers with the request/response being there to analyse.

Later, when we decided to contribute the change upstream, it somehow didn't seem right to include `debug_output` in the SDK as it was a convenience to see _verbose_ request/response output but something not required too often.

So, how can we get the desired convenience without making the change in [razorpay-ruby](https://github.com/razorpay/razorpay-ruby) and at our free will.

Well, we will use the not-so-recommended beahvior in general but useful here. We will re-open the class in our RoR app and enhance the `Razorpay::Request` class to include `debug_output $stdout`. It seems fit to have the change only for `development environment` or as desired.

```ruby
# config/environments/development.rb

config.to_prepare do
  Razorpay::Request.class_eval do
    debug_output $stdout
  end
end
```

> Always strive to get the job done with unnecessary communication.

Happy Debugging !!!
