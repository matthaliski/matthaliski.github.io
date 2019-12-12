---
layout: post
title:  Customizing the Trix toolbar
date:   2019-12-11 11:35:00 -0700
tags: [rails, trix]
---

I recently added the [Trix Editor](https://trix-editor.org/) to a Rails app I was working on and quickly found that I was going to need to do some modifying to the toolbar buttons. Out of the box Trix only gives you an `h1` button for creating headers and the site I was building was already populating an `h1` tag via a separate field. Therefore, my goal was to remove the `h1` button and add a couple smaller header buttons instead. This is what I was going for.

![Updated Trix Bar](/images/2019-12-11/trix-example.png)

At the time of this writing I don't believe there are any proper hooks to add/remove buttons, but you can still certainly get it done. It should be noted that I'm also using [Stimulus](https://stimulusjs.org/), so it uses that framework to get the functionality up and running.

I'm also making the assumption that you already have Trix setup and running. If not, make sure you get through that first because I'm not covering it here.

## Step 1: Hide the h1

Again, no hooks, so this is as simple as it gets. We're just going to hide it with CSS.

```css
.trix-button--icon-heading-1 {
  display: none;
}
```

## Step 2: Extend formatting attributes

```javascript
Trix.config.blockAttributes.heading = {
    tagName: "h2",
    terminal: true,
    breakOnReturn: true,
    group: false
}

Trix.config.blockAttributes.subHeading = {
    tagName: "h3",
    terminal: true,
    breakOnReturn: true,
    group: false
}
```

This Rails app was also my first foray into using Webpacker. So I put this code in `/app/javascript/application/trix_customization.js`. I won't pollute this post with a bunch of Webpacker stuff. There are a bazillion ways to configure it and every app will have its own needs.

## Step 3: Configure the Stimulus controller

```javascript
// app/javascript/controllers/trix_toolbar_post_controller.js
import { Controller } from "stimulus"

export default class extends Controller {

    connect() {

        // Grab a reference to the toolbar(s) on the page.
        const toolbar = this.element.previousSibling
        // HTML for our buttons
        const h2ButtonHTML = '<button type="button" class="trix-button" data-trix-attribute="heading" title="Subheading">H2</button>'
        const h3ButtonHTML = '<button type="button" class="trix-button" data-trix-attribute="subHeading" title="Subheading">H3</button>'
        // Only apply event listeners once to the toolbars
        const once = {
            once: true
        }

        addEventListener("trix-initialize", function(event) {
            const sibling1 = toolbar.querySelector(".trix-button--icon-increase-nesting-level")
            sibling1.insertAdjacentHTML("afterend", h2ButtonHTML)
            const sibling2 = toolbar.querySelector("[data-trix-attribute='heading']")
            sibling2.insertAdjacentHTML("afterend", h3ButtonHTML)
        }, once)
    }
}
```

Thanks to Stimulus, `connect()` is going to get called if the data attribute `data-controller="trix-toolbar-post"` is found on the page. So make sure you add that to the appropriate `rich_text_area` in your form.

```erb
<%= f.rich_text_area :content, data: { controller: 'trix-toolbar-post' } %>
```

## Final *gotcha*

The Trix JavaScript wants to be global. So when you compile your JavaScript, you're going to need to help it along. My example below won't fit for everyone's setup, but hopefully you'll get the gist of it. 

As previously mentioned, I'm using Webpacker to manage all my JavaScript. That means I use `app/javascript/packs/application.js` as a manifest, of sorts. If you're in a similar situation you probably see something like the following already in that file.

```javascript
require("@rails/ujs").start()
require("turbolinks").start()
require("@rails/activestorage").start()
require("channels")
```

Here are my additions to `app/javascript/packs/application.js` to get Trix setup and included.

```javascript
// Trix wants to be global
window.Trix = require("trix")
// Include ActionText
require("@rails/actiontext")
// Include our button customization
require("application/trix_customization")
// Ensure the app/javascript/controllers directory is included since we're using
// Stimulus
import "controllers"
```

That should get you a couple shiny and new header buttons. I'm sure as the project matures, we'll get actual hooks that allow us to yank out unwanted buttons and add new ones. However, this should get you up and running in the meantime. 