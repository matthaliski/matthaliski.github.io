---
layout: post
title:  How to create a contact form with Mandrill
date:   2016-02-08 05:55:56 -0700
tags: rails email
---

Let's build a simple contact form for your Rails driven website that uses [Mandrill](http://mandrill.com/) as the desired email server. Mandrill lets us offload the management of setting up and running an email server ourselves and, therefore, relieving ourselves of a massive headache. Mandrill starts you with 2,000 free trial sends which is more than enough to decide if you're going to like it.

## Setup a Mandrill account

If you do not have a Mandrill account you'll need to go [here](https://mandrill.com/signup/) and register. Once you've done that and logged into the Dashboard you should click **Settings**. You'll see some information we'll end up using later, but it's important that you generate a new API Key for the project you're working on.

[img](3)

You'll want to generate a new key for each project you're working on. This would allow you to disable or revoke a particular API key without disrupting service on any other sites.

Keep that page open so we can copy over some important info into your environment variables.


## Set your environment variables

I'm going to assume you know what environment variables are and why they're important and good to make use of. You might [start here](https://en.wikipedia.org/wiki/Environment_variable) if you don't know what they are.

I do my development on a Mac and have chosen to put my environment variables into `.bash_profile`. Open whatever file you've chosen to use and enter something similar to the following.

```
## Mandril
export MANDRILL_USER="user@example.com"
export PROJECTNAME_MANDRILL_KEY="..."
```

Save that file and close out of it. You're Mandrill user is going to be the same for every project, but you'll probably end up having multiple API keys. I've chosen to use the pattern `PROJECTNAME_MANDRILL_KEY` to distinguish between the various keys I use.

## Update secrets.yml

You've added your environment variables so now make sure they're referenced in your `secrets.yml` file.

```yaml
common: &common
  mandrill_user: <%= ENV['MANDRILL_USER'] %>
  mandrill_key:  <%= ENV['PROJECTNAME_MANDRILL_KEY'] %>

development:
  <<: *common
  secret_key_base:blahblah

test:
  <<: *common
  secret_key_base:blahblah

production:
  <<: *common
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>

```

I typically have a `common` group that can be shared across environments. This *does* mean we're using the same Mandrill API Key across all the environments. You may wish to create separate keys.

## Configure Action Mailer

Open the file `config/environments/development.rb` and add the following with your own values.

```ruby
...
# ActionMailer Config
config.action_mailer.default_url_options = { host: 'localhost:3000' }
config.action_mailer.delivery_method = :smtp
# change to true to allow email to be sent during development
config.action_mailer.perform_deliveries = true
config.action_mailer.raise_delivery_errors = true
config.action_mailer.default :charset => "utf-8"
config.action_mailer.smtp_settings = {
    address:    "smtp.mandrillapp.com",
    port:       587,
    user_name:  Rails.application.secrets.mandrill_user,
    password:   Rails.application.secrets.mandrill_key
}
...
```

These are just simple configs. Save and close. We'll do nearly the same for `config/environments/development.rb`.

```ruby
...
# ActionMailer Config
config.action_mailer.default_url_options = { host: 'https://example.com' }
config.action_mailer.delivery_method = :smtp
config.action_mailer.perform_deliveries = true
config.action_mailer.raise_delivery_errors = false
config.action_mailer.default :charset => "utf-8"
config.action_mailer.smtp_settings = {
    address:    "smtp.mandrillapp.com",
    port:       587,
    user_name:  Rails.application.secrets.mandrill_user,
    password:   Rails.application.secrets.mandrill_key
}
...
```

The differences between development and production are:

* `default_url_options` - Change to your production URL.
* `raise_deliver_errors` - Change to **false**

## Create the Message model

We're going to create a Plain Old Ruby Object, or PORO, and include a few classes so we can benefit from some ActiveModel features and not reinvent the wheel. We're not going to use the generator to create the model. Instead, create the file `app/models/message.rb`.

```ruby
class Message
  include ActiveModel::Model
  include ActiveModel::Conversion
  include ActiveModel::Validations

  # Let's us access as if they were actual database objects
  attr_accessor :name, :email, :subject, :message

  # Yay validation!
  validates :name, :email, :subject, :message, presence: true
end
```

## Create the MessageMailer

Now we move on to generating our `MessageMailer`. Run the following from your Terminal:

```
rails generate mailer message_mailer
```

After that completes, open `app/mailers/message_mailer.rb` and add the following:

```ruby
default from: "Project Message <messages@example.com>"
default to: "recpient@example.com"

def contact_team(message)
  @message = message.message
  @subject = message.subject
  @from = "#{message.name} <#{message.email}>"

  mail(subject:@subject, from:@from) do |format|
    format.text
    format.html
  end
end
```

If you're going to require a *from email* on your form, then it's not necessary to have a default from address. In this case, I've included it as a backup. The `contact_team` is a method we'll call from a controller in the next step.

It's important to take note of the formats we've defined. For example, `format.html` and `format.text`. This allows us to send and receive multi-part emails. When we choose to do this we get a little Rails magic allowing us to create two files and have them automagically used. Create the following files with their content:

```erb
<!-- app/views/message_mailer/contact_team.html.erb -->
<p>New application submission</p>
<br>
<p><b>Subject:</b></p>
<%= @subject %>
<p><b>Message:</b></p>
<%= @message %>
```

```erb
<!-- app/views/message_mailer/contact_team.text.erb -->
New application submission

Subject:
<%= @subject %>
Message:
<%= @message %>
```

If it's not obvious, these are our mailer views. I'm not going to focus on adding styles to these emails.

## Create the messages controller

We need to create a controller for our messages. So, go ahead and run this from your terminal:

```
rails generate controller messages
```

After that has run, open `app/controllers/messages_controller.rb`. We need to create two actions and they look like this:

```ruby
class MessagesController < ApplicationController

  def new
    @message = Message.new
  end

  def create
    @message = Message.new(message_params)

    respond_to do |format|
      if @message.valid?
        format.html {
          MessageMailer.contact_crew(@message).deliver_now
          redirect_to contact_path, notice: "Your message has been sent."
        }
      else
        format.html {
          flash.now[:error] = "Oops! We weren't able to deliver your message."
          render :new
        }
      end
    end
  end

  private

  def message_params
    params.require(:message).permit(:name, :email, :subject, :message)
  end

end
```

This will provide us the capabilities for both a `new` and `create` view with the validation goodness we're used to.


## Contact page view

We've done all the behind the scenes work and can now show it off. Create the file `app/views/messages/new.html.erb`. It should look something like the following simple form. *Uses Bootstrap CSS*.

```erb
<h1>Contact Form</h1>
<%= form_for @message, url: contact_team_path do |f| %>
  <div class="row">
    <div class="col-sm-10 col-sm-offset-1 col-md-8 col-md-offset-2">
      <%= render 'shared/form_errors', object: f.object %>
      <fieldset class="form-group">
        <%= f.label :name %>
        <%= f.text_field :name, class:'form-control' %>
      </fieldset>

      <fieldset class="form-group">
        <%= f.label :email %>
        <%= f.text_field :email, class:'form-control' %>
      </fieldset>

      <fieldset class="form-group">
        <%= f.label :subject %>
        <%= f.text_field :subject, class:'form-control' %>
      </fieldset>

      <fieldset class="form-group">
        <%= f.label :message %>
        <%= f.text_area :message, class:'form-control', rows:7 %>
      </fieldset>

      <fieldset class="form-group text-xs-right">
        <%= f.submit 'Send', class:'btn btn-primary' %>
      </fieldset>
    </div>
  </div>
<% end %>
```

## Routes

The last step is to add the appropriate routes to our `config/routes.rb` file.

```ruby
...
# Contact the team
  match '/contact-team', to:'messages#new', via: :get, as:'contact'
  match '/contact-team', to:'messages#create', via: :post
...
```

## That's it!

That completes the process of setting up a contact form in Rails using Mandrill. It took a while, but we got there ;)


#### Other considerations

This is a bare-bones example that doesn't deal with potential abuse from evildoers. It would probably be prudent to, among other things, ensure the form was rate-limited so a user couldn't spam you with thousands of messages all at once. That's outside the scope of this log entry, but could use consideration.
