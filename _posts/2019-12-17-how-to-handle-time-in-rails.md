---
layout: post
title:  How to handle time in Rails
date:   2019-12-17 9:53:00 -0800
tags: [rails]
---

Handling time is a doozy. It's not necessarily a problem that is specific to Rails either. It's just hard. There are a thousand things to consider. I want to help you get an idea of what a reasonable setup might look like and let you go from there. The following steps are meant to be more conceptual than a line-by-line tutorial. It's more important that you grasp the big stuff rather than just copy and paste.

## What we're up against

Our visitors are jumping on our websites from all over the world. This prohibits us from displaying a simple and static time for our records. Each visitor likely needs to see the time reflected in their respective timezone.

We face a similar, but more nuanced, issue in our admin section. Let's imagine we're a news organization which has reporters dispersed throughout the U.S. In order to meet deadlines, these reporters need to ensure the timezone they're dealing with is known and fixed. This ensures they can meet a, let's say, midnight deadline in the agreed upon timezone. Something like `"Eastern Time (US & Canada)"` rather than the physical location they may be publishing from. That means we need to let our reporters set their own preferred timezones on some sort of settings screen.

Lastly, in order to do any of this correctly we want to make sure we always save as [`UTC`](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) to the database. Here's a diagram to help visualize.

![Time diagram](/images/2019-12-17/time-diagram.png)

If we can accomplish this we'll be off to a great start. Our strategy is essentially this:

1. Admin section - convert time based on `time_zone` field in the `User` record
2. Database - stick with Rails defaults so everything is UTC
3. Frontend display - use [Local Time gem](https://github.com/basecamp/local_time) (by Basecamp)

## Configuring Rails

Let's start with making sure we have Rails set up correctly. We want this correct from the start because if this part gets borked we can kiss the rest of our efforts goodbye. If you're in the initial stages of setting up your site and you haven't explicitly set any time configs, then you're in great shape. Rails is going to assume you want everything in `UTC`. If you're unsure how your app is set up you might want to look in `config/application.rb`. If you see

```ruby
# Don't set this here.
config.time_zone = "Eastern Time (US & Canada)"
```

or any other explicit timezone set, you're going to want to clear that out. If you were operating a small intranet app at a single location then this setting might be a viable solution. However, if your website is live to the world you probably don't want to set this config in this manner.

Great, by essentially doing nothing we've accomplished #2 from above. Everything gets saved as `UTC`, which is what we want.

## Dealing with the admin section

Think back to our news organization analogy. We need to let each of our reporters set their timezone on some sort of settings screen. Therefore it makes sense to run a migration on our `User` model (or whatever you call yours) to add a `time_zone` field. It could look something like this:

```ruby
class AddTimezoneToUser < ActiveRecord::Migration[6.0]
  def change
    add_column :users, :time_zone, :string, null: false, default: 'Pacific Time (US & Canada)'
  end
end
```

Now that we have that field available to us, we can do something pretty cool. We can use a filter to set the timezone for our logged in reporters. We'll do this with an `around_action` in our `application_controller.rb`

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  around_action :set_time_zone, if: :current_user

  ...

  private

  def set_time_zone(&block)
    Time.use_zone(current_user.time_zone, &block)
  end
end
```

This basically says, "If there is a `current_user` signed in, use the `time_zone` from their profile when displaying time." This is great because if we display something like this in a form

```erb
<%= f.datetime_field :published_at, class: 'form-control' %>
```

we end up getting Rails to convert the UTC time saved in the database to whatever timezone the user saved in their profile. And the conversion happens both ways. If you update the time in the form it will be converted back to UTC prior to hitting the database.

Hey, we're doing great. We've solved our first two goals from above.

## Showing visitors time on the frontend

Our last task is to solve the problem of showing time to visitors who won't be logged into our website. In other words, we don't know who they are or where they are. Competently solving this problem, I imagine, would be quite the undertaking. My brain starts to hurt when I think of the variables involved. That is why it's great news that the wonderful folks at [Basecamp](https://basecamp.com/) already solved this problem for us with a gem called [Local Time](https://github.com/basecamp/local_time).

I don't typically subscribe to a strategy of throwing gems at everything I need to solve, however I feel pretty good about using something from Basecamp. If you're unfamiliar with the history of Rails, then you'd probably be interested to learn that much of what we have in Rails today has been ported out of Basecamp and integrated as core features. The two projects are like conjoined code twins. Long story short, I think we're safe adding this gem to our project.

```ruby
gem 'local_time'
```

Give it a `bundle install`. The gem is going to use JavaScript to perform some of the detection needed to acquire the visitors timezone, so make sure you include it via the Asset Pipeline or Webpacker.

Now we can use the gem's helper to easily display our timezone sensitive field to our unknown visitors:

```erb
<%= local_time(post.created_at) %>
```

## Conclusion

Hopefully that gives you the philosophy of handling timezones in Rails. The examples given should be loose enough to fit into your existing projects. You may also have additional needs that reach beyond this, but you'll have a good foundation to work from now.

Happy Holidays! I need to go wrap some presents while the fam is out of the house.
