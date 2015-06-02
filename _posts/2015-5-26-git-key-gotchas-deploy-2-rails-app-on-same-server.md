---
layout: post
title: Git Key Gotchas - Deploy 2 Rails App on Servers 
---

Its inevitable to avoid having No Api's for a big Business.

Mobile presence comes chasing and forces to have an Api.

Api's are fun but why it is sensed otherwise above.

Yeah, you guessed it right, its not the Api but documentation which is
pain yet most important.

Starting from sharing in Google docs to using [documentation
gem](https://github.com/adamcooke/documentation) to
change / update at will is a step in right direction.

[documentation
gem](https://github.com/adamcooke/documentation) is compatible with Rails 4 and has hell lot of
dependencies and that too very different from our Rails 3 App.

To avoid burdening our Ruby Process with gems that we occasionally use
and ofcourse the incompatibility with rails, we decided to create a new
Rails 4 App just for documentation.

Yep, an App serving the documentation only as its so-so important :)

Being new, we thought we have all the control and would be a child's
play to make it available for creating documentation.

We got it working on local in no-time. Now, we have to make it available
to testers and internal developers so that they can use and start the
dance of including their Api documentation.

Generally, we deploy single App to one server. But for such trivial
application, we considered creating a new server and deploy there a big
waste.

So we decided to deploy staging App on one of test Servers and final
updated documentation on one of the production servers.

But as always, developer endeavours never last so smooth and here it
came :P

We deployed on Test Server with confidence and first deploy step failed:

*Syncing Code from Github*

It looked easy as we can add the deploy keys to the documentation Github
Repo and it would work.

As same Deploy key was getting used for other application on same
server, Github complained of key already in use.

Such simple problem became complicated as without code being on server,
nothing is fun :(

As all good developers when in trouble, we looked at Google for help :P

Solution was right there in Github Help docs and there were 4.

Also, kinda same problem is commonly faced with Heroku when you have
personal and company projects, it too complains of key in use if you
attempt to put same key in both.

We set out on 2 diff. paths to figure what can be easy and would work
for us among 4 solutions proposed by Github and Heroku challenge of keys
b/w personal and company projects.

Solution 1 - [Use ssh-forwarding](https://developer.github.com/guides/using-ssh-agent-forwarding/)
======

Only change needed is <On Local System - Deploy is attempted>

- Open / Create `~/.ssh/config`
- Add

```
Host staging-server.com
  ForwardAgent yes
```
- Save
- ssh-add -L
  - should show Identity present
  - If not, use `ssh-add <key>`
  - <key> is generally id_rsa, if nothing fancy tried with SSH
  - Try `ssh-add -L` again, should show Identity present
- Continue with deploy command
- Works like a charm

Solution 2 - [Use different SSH key](http://stackoverflow.com/questions/7927750/specify-an-ssh-key-for-git-push-for-a-given-domain) [1](http://superuser.com/questions/366649/ssh-config-same-host-but-different-keys-and-usernames)
=====

- Create a key on server using ssh-keygen -T
- Make sure it's differently named than default `id_rsa`
  - let's call `new_rsa` for further references
- Add the `new_rsa` key as Deploy Key in github project 
- Add an entry in `~/.ssh/config`

```
Host server.work
  HostName heroku.com
  Username dummy
  IdentityFile "/Users/john/.ssh/identity.heroku.personal"
  IdentitiesOnly yes
```
- Change in deploy file <cap setup assumed for deploy>
  - set :repo_url, 'git@server.work:dummy/demo_app.git'
- *Note* `server.work` instead of `github.com`
- cap3 adds a repo directry in deploy directory at server
  - it might need a change in git config remote.origin.url from
    git@github.com:dummy/demo_app.git to git@server.work:dummy/demo_app.git 

*Solution 1* is easy to set-up and changes required only on local file but
forces to trigger the deploy command from local machine manually <No
automatic deploys>

*Solution 2*, though more configuration can be used to deploy from server
directly and can be automated

Taming the DevOps !!
