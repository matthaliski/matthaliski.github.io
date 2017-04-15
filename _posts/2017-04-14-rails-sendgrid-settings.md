---
layout: post
title:  ActionMailer SMTP settings for SendGrid (Rails)
date:   2017-03-30 05:55:56 -0700
categories: coding, rails
---

Okay, I got no Google hits for this. Here's a tip if you're using SendGrid in a Rails app for your transactional emails. A while back you needed to provide your SendGrid username and password (hopefully via environment variables), but now you can use an app-specific api key.

Here's some historical context. This was the old way:

```ruby
config.action_mailer.smtp_settings = {
  address:        "smtp.sendgrid.net",
  port:           587,
  domain:         "yourwebsite.com",
  authentication: :plain,
  user_name:      Rails.application.secrets.sg_user_name
  password:       Rails.application.secrets.sg_password
}
```

So, this basically sucked because you were potentially providing your SendGrid username and password to a whole bunch of apps. What happens when you need to revoke a rogue app? If you change your password you bork _ALL_ your other apps using that account.

Now, you can just grab an apikey in the SendGrid settings and do this:

```ruby
config.action_mailer.smtp_settings = {
  address:        "smtp.sendgrid.net",
  port:           587,
  domain:         "yourwebsite.com",
  authentication: :plain,
  user_name:      'apikey',
  password:       Rails.application.secrets.sendgrid_api_key
}
```

Notice the **apikey** username. That was the key that I couldn't find documented anywhere on their site. Now, when you need to revoke access for that particular app you just kill the apikey. Your login credentials to SendGrid are safe.

Enjoy your weekend.
