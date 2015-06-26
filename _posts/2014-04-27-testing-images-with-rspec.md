---
layout: post
title: Testing images with RSpec
date: 2014-04-27 11:58:38.000000000 +03:00
tags:
- image
- RSpec
- Ruby on Rails
---

To test whether the right image is displayed on a page:

{% highlight ruby %}
expect(page).to have_css("img[src*='image_name.png']")
{% endhighlight %}

i.e. "Expect page to have an `<img>` tag whose `src` attribute contains the name of the required image."
