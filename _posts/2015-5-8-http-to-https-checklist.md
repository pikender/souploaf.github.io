---
layout: post
title: Http to Http(s) Checklist 
---

To enter the mobile space, we created an Android App.

Everything finished with not much challenges and tested properly for
good user experience.

Being contended, we started testing on Adroid 5.x series and Boom!!, the
payment redirection in web view came crumbling.

On inspection, it was concluded that "as payment server is https so
Android Web view accepts it to redirect to an https url instead of http
as in our case"

We released the App knowing the bug and proposed resolution

*Implement Https*

as only 1% of Android Devices were 5.x and most of the users aggregated
in US and thankfully not our target Market :)

We started with the obvious step of buying a certificate from one of the
Vendors.

[Followed the steps to generate needed certificates to put on servers
and in nginx config](http://www.akadia.com/services/ssh_test_certificate.html)

Now site can be accessed using https, theoretically.

In our case, we had mix of http and https, https only on cart and other
http (listing pages) for performance reasons.

Above constraint made it bad and better for different reasons.

*Bad*
- Handle Mix content warnings

*Better*
- Learnings to handle such cases :)

To abstract the basic requirement of redirection to https when accessed
through http for secure pages and allowing both http / https in some
cases, we resorted to [Gem
ssl_requirement](https://github.com/bartt/ssl_requirement)

It helped us handle above requirement declaratively through
*ssl_allowed* and *ssl_required* helpers

Next we had to resolved assets falling to 2 major categories:

- CSS / Javascript
- Images

As CSS / Javascript implies presence of Browser, which is smart enough
to figure relative urls namely *//assets.example.com* and prepend them
with protocol of page its rendering, automatic prepend of http / https.

Bingo !!

We got lucky in images too as they were hosted on Amazon S3 which
supports both http / https for same resource.

We took an easy path to always pick https on both http / https pages, as
https pages were getting https so no complaints of Mixed Content and
http was obliged to be treated for https content :)

Note: https was just a single alphabet change in fog_host config of s3
in paperclip gem, you guessed right the letter was "s in https"

Now, Application was tuned for smooth sail to get it rolling for all.

To handle load, we are having multiple App Servers behind a load
balancer.

Our Load Balancer ditched us by behaving abnormally when both http /
https routed through it mainly POST requests, if it happens to route to
same server it came from, it behaved properly but which we can't assure
being load balanced.

As an alternative, we decided to use 2 Load Balancers with shared IP,
one catering to Https requests by only listening to Port 43 and other
for Http.

Above Setup worked like a charm, as HTTPs pages got rendered / routed
through same Load balancer always.

Oh Wait, we didn't mentioned anything about App.

After all the juggling, we weren't very sure of everything working
smoothly not because it won't work but to have a contingency / backup
plan if it didn't :)

We introduced a configuration, which converts urls to http / https as
needed and controlled its change through an Api on Server.

Rollbacks / Deploys are easy on Web but not App as it requires to go
through a confirmation process which could take hours to days.

Summary
=======

- [Steps to Generate Certificates for Nginx / Servers](http://www.akadia.com/services/ssh_test_certificate.html)
- [ssl_requirement](https://github.com/bartt/ssl_requirement)
- Relative Urls for CSS / Javascript
- Images Always Https
- Shared Load Balancer (One for http and othet for https)
- App control of http / https Urls based on Server Config controlled
  through Api
