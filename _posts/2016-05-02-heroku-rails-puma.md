---
layout: post
title:  Heroku defaults to Puma for Rails 5
date:   2016-05-02 05:55:56 -0700
categories: rails
redirecturl: https://leavingharbor.com/posts/heroku-defaults-to-puma-for-rails-5
---

It's now going to be easier than ever to kick off a new Rails project and deploy
to Heroku. [WEBrick][1] is no longer the default server with Rails 5. If you don't
provide a `Procfile` Heroku is going to help you out and set you up with Puma.
Production ready out of the box. So... that's cool.

Also of note, configuration is going to default to matching Puma threads to Active
Record connection pool size. Helpful for avoiding connection timeout errors.

They're doing some other cool things and you can [check their post][2] for all the
details

[1]: http://ruby-doc.org/stdlib-2.3.0/libdoc/webrick/rdoc/WEBrick.html
[2]: https://blog.heroku.com/archives/2016/5/2/container_ready_rails_5
