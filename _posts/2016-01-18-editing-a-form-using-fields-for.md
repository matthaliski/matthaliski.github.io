---
layout: post
title:  Editing a form using fields_for in Rails
date:   2016-01-18 05:55:56 -0700
categories: rails
redirecturl: https://leavingharbor.com/posts/editing-a-form-using-fields_for-in-rails
---

Let's say you'd like to successfully create or update a nested form using `fields_for`. Our 'parent' will be a Post model and our 'child' will be a Tag model. It's not hard, but there's a bit of a *gotcha* that I found most other resources out there were leaving out.

## Post (our parent)

We'll use a Post model as our parent in this situation. Nothing out of the ordinary here. A post can have many tags and we'll allow them to be destroyed if the model itself is.

```ruby
# app/models/post.rb

class Post < ActiveRecord::Base
  has_many :tags, dependent: :destroy

  # Enable the building of complex forms
  accepts_nested_attributes_for :tags, allow_destroy: true
end
```

It's important to include `accepts_nested_attributes_for ` to allow you to save attributes on the associated Tag model. You get no Rails magic if you forget this.

## Tag (our child)

Again, nothing special here. Just showing that it belongs to our post model

```ruby
# app/models/tag.rb

class Tag < ActiveRecord::Base
  belongs_to :post
end
```

## The Controllers

Now we get to some of the important stuff. There are a few things to take note of in the new and edit actions.

```ruby
# /app/controllers/posts_controller.rb
def new
  @post = Post.new
  # Ensure we have a blank tag to start with
  1.times { @post.tags.build }
end

def edit
  @post = Post.find(params[:id])
  1.times { @post.tags.build }
end

private

def post_params
  params.require(:post).permit(:title, :body, tags_attributes:[:id, :title])
end
```

So, a few things here.

* We need to build an empty `Tag` object with `1.times { @post.tags.build }` so the our form view doesn't bork on us.
* Get our Strong Parameters correct. It's probably obvious you would need to permit the `:title` nested attribute for `tags_attributes`, but perhaps not so obvious you should include `:id`. After all, we don't explicitly include `:id` for our parent model.

## The View

We're going to follow the common pattern of sharing a `_form.html.erb` partial between the *new* and *edit* views.

```erb
<% # app/views/admin/posts/_form.html.erb %>

<%= form_for [:admin, @post] do |f| %>

  <%= render '/shared/form_errors', object: f.object %>

  <fieldset>
    <%= f.text_field :title, class:'form-control', placeholder:'Title' %>
  </fieldset>

  <fieldset>
    <%= f.text_area :body, class:'form-control', placeholder:'Body', rows:10 %>
  </fieldset>

  <h3>Post Tags</h3>
  <%= f.fields_for :tags do |tag_form| %>
    <fieldset>
      <%= tag_form.text_field :title, class:'form-control' %>
    </fieldset>
  <% end %>

  <fieldset>
    <%= f.submit "Save", class:'btn' %>
  </fieldset>

<% end %>
```

Now it should be obvious why we had to build an empty `Tag` object. If we hadn't, we would have an error with `tag_form.text_field :title` when on the *new* view because `tag_form` would be `null`.

The fact that we're using `1.times { @post.tags.build }` in the `edit` action means we'll always have an extra blank field should we want to add an additional tag.

## Summary

Creating nested forms in Rails isn't overly complicated. I just got hung up on the necessary Strong Parameters to permit, namely the nested `:id`.
