---
layout: post
title: 3rd Party Integration using Rails Engines 
---

### What are Rails Engines ?

[Engines](http://guides.rubyonrails.org/engines.html) can be considered miniature applications that provide functionality to their host applications

### Popular Rails Engines

- [Devise](https://github.com/plataformatec/devise) , an engine that provides authentication for its parent applications, or 
- [Forem](https://github.com/radar/forem) , an engine that provides forum functionality.
- [Spree](https://github.com/spree/spree) which provides an e-commerce platform, and
- [RefineryCMS](https://github.com/refinery/refinerycms), a CMS engine.
 
### What is 3rd Party ?

Anything which we don't own and are using is 3rd Party

Typical Examples are Payment Gateways, Mailing / Sms services in
e-commerce app.

Some classify as any service as 3rd Party

### Now familar with some of the typical examples of 3rd Party and Rails Engines, 

- lets dive deep into the co-relation between 3rd Party Integration &
  Rails Engines and
- how can we leverage Rails Engines to fully isolate 3rd Party
  Integration from Core of App
- Why it could be a bad idea to directly jump to Rails Engines whenever
  3rd Party Integrations come along

### We will be starting with the incremental process of extraction

1. Starting with 3rd Party Integration code in our Rails App
2. Extracting the stable parts not interacting with request / response
  cycle into a [Ruby Gem](http://guides.rubygems.org/what-is-a-gem/)
[Wiki](https://en.wikipedia.org/wiki/RubyGems)
3. Extracting the parts interacting with request / response
  cycle into a [Rails Engine](http://guides.rubyonrails.org/engines.html)

Incremental Process of Extraction is an important bit as systems running
and used daily can never be changed in a day and requires extra effort
to understand the feedback / connection with other parts in system.

Extract a part and hook it back with defined interface, let it stabilize
and then start with next bit.

From the looks of it, you must have guessed it by now that the described
activity is a
[REFACTORING](https://en.wikipedia.org/wiki/Code_refactoring) EXERCISE.

### Brief Introduction about the extraction we would do

There is a payment gateway by the name of Gtpay in Nigeria.

For better or worse, there is no Ready-made Ruby Gem available for its
integration like Stripe / Paypal

We started with integrating it in our app and now when its running
smoothly, we decided to decouple it from our app.

Couple of Payment Integrations are already integrated and thrown away.

In an attempt of not loosing what we created and also, keeping it very
easy to include / exclude when needed, we started the journey from RAILS
APP to RUBY GEM to RAILS ENGINE

### Process Walkthrough of what we would extract step - by - step

- A link is needed to process the payments through Gtpay
<img src="/images/Gtpay1.png" alt='Click Gtpay Link'>
- A spinner should come up to indicate connection attempt with Gtpay
  - Behind the scenes (spinner), form need to be submitted to redirect to
  Gtpay with information like transaction amount, currency, merchant
details etc

<img src="/images/Gtpay2.png" alt='Showing Spinner and Connecting'>

- When on Payment Page i.e. Gtpay

<img src="/images/Gtpay3.png" alt='Gtpay Payment Page'>

  - there should be an end-point acting as callback to handle the success / failure responses from Gtpay

<img src="/images/Gtpay4.png" alt='Handle Callack Show Error'>

### Code Structure

#### Monlith Rails App

```sh
app/controllers/
├── application_controller.rb
├── gtpay
│   └── transactions_controller.rb
└── home_controller.rb
app/gtpay/
├── gtpay
│   ├── request.rb
│   ├── response.rb
│   └── transaction.rb
└── gtpay.rb
app/models/
├── gtpay
│   └── gt_pay_transaction.rb
├── order.rb
└── user.rb
app/views/
├── gtpay
│   └── transactions
│       ├── auto_submit_gtpay_forms.html.erb
│       └── create.html.erb
├── home
│   ├── index.html.erb
│   └── new.html.erb
└── layouts
    └── application.html.erb
```

Everything scoped under gtpay is interesting and is a contendor for
extraction

#### Monlith Rails App Reduced and Supported by Gem

##### Gtpay GEM - Business Logic Extracted

```sh
gtpay
├── Gemfile
├── Gemfile.lock
├── README.md
├── Rakefile
├── gtpay.gemspec
├── lib
│   ├── generators
│   │   ├── gtpay
│   │   │   ├── controllers_generator.rb
│   │   │   ├── install_generator.rb
│   │   │   └── model_generator.rb
│   │   └── templates
│   │       ├── controllers
│   │       │   └── transactions_controller.rb
│   │       ├── initializers
│   │       │   └── gtpay.rb
│   │       ├── migrations
│   │       │   └── 001_create_gtpay_transaction.rb
│   │       ├── models
│   │       │   └── gt_pay_transaction.rb
│   │       ├── spec
│   │       │   └── gtpay_transactions_controller_spec.rb
│   │       └── views
│   │           └── transactions
│   │               ├── auto_submit_gtpay_forms.html.erb
│   │               └── create.html.erb
│   ├── gtpay
│   │   ├── request.rb
│   │   ├── response.rb
│   │   └── transaction.rb
│   └── gtpay.rb
└── spec
    ├── active_record
    │   ├── setup_ar.rb
    │   └── test.db
    ├── gtpay
    │   ├── gtpay_spec.rb
    │   ├── request_spec.rb
    │   ├── response_spec.rb
    │   └── transaction_spec.rb
    └── spec_helper.rb
```

##### Rails App - Gem got included back in Gemfile

```ruby
# Gemfile
gem 'gtpay', '0.0.1'
```

```ruby
# gtpay_engine.gemspec
s.add_dependency "gtpay", "~> 0"
```

```sh
app/controllers/
├── application_controller.rb
├── gtpay
│   └── transactions_controller.rb
└── home_controller.rb
app/gtpay/ [error opening dir]
app/models/
├── gtpay
│   └── gt_pay_transaction.rb
├── order.rb
└── user.rb
app/views/
├── gtpay
│   └── transactions
│       ├── auto_submit_gtpay_forms.html.erb
│       └── create.html.erb
├── home
│   ├── index.html.erb
│   └── new.html.erb
└── layouts
    └── application.html.erb
```

**Rails Engine**

```ruby
# Gemfile
gem 'gtpay', '0.0.1'
```

```sh
gtpay_engine_gem/
├── Gemfile
├── Gemfile.lock
├── MIT-LICENSE
├── README.rdoc
├── Rakefile
├── app
│   ├── assets
│   │   ├── images
│   │   │   └── gtpay_engine
│   │   ├── javascripts
│   │   │   └── gtpay_engine
│   │   │       └── application.js
│   │   └── stylesheets
│   │       └── gtpay_engine
│   │           └── application.css
│   ├── controllers
│   │   └── gtpay_engine
│   │       ├── application_controller.rb
│   │       └── transactions_controller.rb
│   ├── helpers
│   │   └── gtpay_engine
│   │       └── application_helper.rb
│   ├── mailers
│   ├── models
│   │   └── gtpay_engine
│   │       └── gt_pay_transaction.rb
│   └── views
│       ├── gtpay_engine
│       │   ├── application
│       │   │   └── home.html.erb
│       │   └── transactions
│       │       ├── auto_submit_gtpay_forms.html.erb
│       │       └── create.html.erb
│       └── layouts
│           └── gtpay_engine
│               └── application.html.erb
├── config
│   ├── initializers
│   │   └── gtpay.rb
│   └── routes.rb
├── db
│   └── migrate
│       └── 20141114193621_create_gtpay_transaction.rb
├── gtpay_engine-0.0.1.gem
├── gtpay_engine.gemspec
├── lib
│   ├── gtpay_engine
│   │   ├── engine.rb
│   │   └── version.rb
│   ├── gtpay_engine.rb
│   └── tasks
│       └── gtpay_engine_tasks.rake
├── script
│   └── rails
├── spec
│   └── controllers
│       └── gtpay_transactions_controller_spec.rb
└── test
    ├── dummy <Rails App to test Engine>
```

**Rails App - Engine got included back in Gemfile**

```ruby
# Gemfile
gem 'gtpay_engine', '0.0.1'
```

```ruby
# config/routes.rb
mount GtpayEngine::Engine => "/gtpay_engine"
```

```sh
app/controllers/
└── application_controller.rb
app/gtpay/ [error opening dir] <No gtpay folder at all>
app/models/
app/views/
└── layouts
    └── application.html.erb
```

Incremental directory structures are self-explanatory of the movement
and separation / reduction in the host Rails App.

Running / Using the 3 implementations are exactly same as no external
 behavior changed but just code is moved around for better separation
and maintenance, easy to plug in and out of Rails App.

### Benefits

The process of including / excluding a feature as important as
integrating a payment gateway has reduced from many lines of code to
just 2 lines inclusion and removal

```ruby
# Gemfile
gem 'gtpay_engine', '0.0.1'

# config/routes.rb
mount GtpayEngine::Engine => "/gtpay_engine"
```
