---
layout: post
title:  Setting Up Goto Declaration in Atom
date:   2017-03-30 05:55:56 -0700
categories: coding
---

#### Update (screw this)

I'll leave everything below for someone who loves using Atom and is okay have having partial support for _goto_ functionality. I basically want the following (ST3), but can't achieve it in Atom.

![Sublime example][7]

I've tried a million different tweaks and used every package I can find, but I can't get it to work the way it should. If you're a ctags ninja you might have better luck than me.  `¯\_(ツ)_/¯`

___

If you've used some sort of _goto_ functionality before, you'll probably feel a little underwhelmed when you fire up [Atom][1] for the first time. I've done all the banging my head against the desk for you. Here's how you get it done.

## What is _goto_ declaration?

I primarily do my web projects in Rails. So, let's take an example from that framework. Let's say I have a goofy helper:

```ruby
def site_title(title)
  title.titleize
end
```

Let's also say that I'm lazy or rushed and I buried this helper in an unknown file. Later, I come across its usage in a view:

```html
<%= content_tag :h1, site_title('this page is awesome') %>
```

I see that a helper is being called, but I can't remember where it is. Maybe I didn't even write the helper to begin with. You could `cmd+shift+f` to search for it, but that's cumbersome. In RubyMine you'd be able to `cmd+click` the helper and it would open the file where the helper is declared and put the cursor on that line. Sublime Text 3 has built-in functionality as well. But, what about Atom?

Not so much.

## Configuring Atom

As of this writing, Atom ships with a package called [symbols-view][2]. This will give you some keybindings and a _Go to Declaration_ option in your right-click (or cmd+click) menu. However, it won't do a damn thing for you without some configuration.

Symbols-view requires the presence of [ctags][3]. I won't get too much into ctags, but they're essentially an index file that maps symbols. You're going to need a way to generate a `.tags` file for this all to work.

### Install the symbol-gen package

In order to generate a `.tags` file install [this package][4]. Once it's installed, you can use `cmd+alt+g` to generate the `.tags` file in the root of your project.

If the computer gods are looking down upon you, you'll now be able to use the symbols-view keybindings or context menu commands. It will look something like this:

![Goto declaration][5]

Victory! But wait, there's more...

### Configure ctags for CSS

Completing the steps above will help you with a bunch of the ruby-specific stuff. However, if you dream of being able to go to the declaration of a CSS class from an `.html.erb` file you're currently boned. You need to tell ctags to include CSS.

Open up `~/.ctags` and add the following:

```
--langdef=css
--langmap=css:.css
--langmap=css:+.scss
--langmap=css:+.sass
--langmap=css:+.styl
--langmap=css:+.less
--regex-css=/^[ \t]*(([A-Za-z0-9_-]+[ \t\n,]+)+)\{/\1/t,tag,tags/
--regex-css=/^[ \t]*#([A-Za-z0-9_-]+)/#\1/i,id,ids/
--regex-css=/^[ \t]*\.([A-Za-z0-9_-]+)/.\1/c,class,classes/
```

Credit to [this post][6] for the ctags snippet. After you add the above and save the file you can generate your `.tags` file again by using `cmd+alt+g`. This should now give you some CSS functionality. It can find symbols, but no goto functionality :(



[1]:https://atom.io/
[2]:https://atom.io/packages/symbols-view
[3]:http://ctags.sourceforge.net/whatis.html
[4]:https://atom.io/packages/symbol-gen
[5]:/images/posts/20170330/goto-declaration.gif
[6]:http://ellengummesson.com/blog/2014/07/27/css-ctags/
[7]:/images/posts/20170330/sublime-example.gif
