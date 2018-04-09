---
layout: post
title:  Syntax Highlighting in Rails
date:   2016-02-01 05:55:56 -0700
categories: rails
redirecturl: https://leavingharbor.com/posts/syntax-highlighting-in-rails
---

Your Rails driven website has the need to display some code. It's not good enough to just get proper indentation, you'd like to have some decent syntax highlighting. Maybe something like following.

```ruby
def foo(bar)
  "Hello #{bar}"
end

foo("World")
```

There's a great ruby gem called [Rouge](https://github.com/jneen/rouge) that makes this process quite easy for us. What is Rouge?

> Rouge is a pure-ruby syntax highlighter. It can highlight over 60 languages, and output HTML or ANSI 256-color text. Its HTML output is compatible with stylesheets designed for pygments.

Pretty cool, let's get going.

## Install Rouge Gem

First, we need to add the Rouge gem to our `Gemfile`

```ruby
# Gemfile  
gem 'rouge', '~> 1.10'
```

After you've added the Rouge gem go ahead and run `bundle install`. Good to go? Okay, let's move on.

## Configuration

There are two parts main parts to the configuration. The first part is the lexing and formatting of your desired code block. The second part is the style (think CSS) you want to give to the now formatted code.

### Lexing and Formatting

Create the initializer `rouge_highlighter.rb` so we can reuse the same formatter and lexer throughout our application.

```ruby
# config/initializers/rouge_highlighter.rb
module RougeHighlighter
  class Highlight
    def initialize
      @formatter = Rouge::Formatters::HTML.new(css_class: 'highlight')
      @lexer = Rouge::Lexers::Shell.new
    end

    def render(source)
      @formatter.format(@lexer.lex(source))
    end
  end

  ::HighlightSource = Highlight.new
end

```

We create a simple Ruby module to house a class and a top-level namespace.

The `@formatter` variable we're setting is going to ensure our provided source code is going to be wrapped in a `.highlight` class. It will wrap our code like this:

```html
<pre class="highlight">
  <code>
    ...
  </code>
</pre>
```

This is important to take note of because we need to provide the `.highlight` scope when dealing with our styling a little later.

We're going to end up using the `render` method in a helper for our application. So, open up `app/helpers/application_helper.rb` and add the following.

```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  ...
  def syntax_highlight(text)
    # Initialized in config/initializers/rouge_highlighter.rb
    html = HighlightSource.render(text)
    html.html_safe
  end
  ...
end

```

Pretty slick. Now we can simply call the `syntax_highlight` method anywhere we want some highlighted code. But, before we do that we need to make sure there is some CSS to back it up.


### Styles (CSS)

In order to have the styles that make all this work look pretty we need to create a new file called `_rouge.scss.erb`. It's a little weird to see `erb` at the end of a `scss` file, but remember Rails interprets from right to left so adding that suffix gives us the some dynamic abilities.

```scss
// app/assets/stylesheets/_rouge.scss.erb
<%= Rouge::Themes::Github.render(:scope => '.highlight') %>

.highlight {
  border-radius:4px;
  background: #f2f2f2;
  margin-bottom:1rem;
  padding:1rem;
}
```

In the first line we've allowed Rouge to spit out some styles while telling it we'd like to *namespace* it to the `.highlight` class. Rouge comes with several predefined themes you can choose from. Here we've chosen a Github-like theme for our syntax highlighting. We also added a little of our own styling as well.

`_rouge.scss.erb` should be compiled automatically if you've stuck with the initial Rails setup in `application.css`

## Yay, We're Setup

Everything should be setup and ready to go now. You can go to any view you'd like and print out some pretty code. I'll put some HTML in an external file I can read from later.

```html
<!-- public/example.html -->
<div class="row">
  <div class="col-sm-3">...</div>
  <div class="col-sm-3">...</div>
  <div class="col-sm-3">...</div>
</div>
```

Then, in an example view I'm choosing to call `highlight.html.erb` I render out our nicely formatted code.

```erb
<% # app/views/static_pages/highlight.html.erb %>

<h1>Syntax Highlighting with Rouge</h1>
<p>Below is a very simple example of some syntax highlighting.</p>

<%= syntax_highlight(File.read(Rails.root + "public/example.html")) %>
```

All done.

## Summary

This example just shows how to get syntax highlighting applied to an external file which, to be honest, isn't extremely useful. It does, however, demonstrate the steps to get going. If you're writing a blog or something similar you probably want something a little more dynamic. If you're a regular Github user you're probably comfortable writing in Markdown. We can easily combine Rouge with [Redcarpet](https://github.com/vmg/redcarpet) and do this all with a Markdown parser. I plan on coming back around and showing how to do that. Until then, enjoy your new highlighted code!

___

### Example Repo

If you want to see everything working together you can clone this [Leaving Harbor repo](https://github.com/leavingharbor/syntax-highlight-example) over on Github and poke around in an actual working environment.
