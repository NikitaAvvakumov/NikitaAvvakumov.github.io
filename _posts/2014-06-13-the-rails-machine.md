---
layout: post
title: The Rails Machine
tags:
- gem
- RSpec
- Ruby on Rails
---
I've been working through - and enjoying - the [EdX/Berkeley CS169 course](https://courses.edx.org/courses/BerkeleyX/CS_CS169.2x/1T2014/info) for the past couple of months. With a bit of a break between the two halves, I thought I'd start a small Rails project of my own. After including `rspec-rails` in the Gemfile, I ran bundle install and then

{% highlight bash %}
$ rails g rspec:install
Could not find generator rspec:install
{% endhighlight %}

This was bad. I naturally blamed the gem first, and spent an awful lot of time digging for an answer. At one point, as a control, I added the `cucumber-rails` gem

{% highlight bash %}
$ rails g cucumber:install
Could not find generator cucumber:install
{% endhighlight %}

This was much worse. The gems were installed and bundled, no errors, no missing dependencies. I went as far as installing the latest Ruby, creating a new gemset, reinstalling Rails, ...

{% highlight bash %}
Could not find generator cucumber:install
{% endhighlight %}

Running `rails g` only showed the built-in Rails generators: model, controller, etc. None of the gem generators would show up. This meant no devise, no foundation, no bootstrap, no hundreds of other gems that I haven't used or that have not even been written yet. I dug deeper, hitting pages 3 and 4 of my Google searches, looking at questions and answers from 2, 3, 4 years ago. Nothing.

It was late. I turned off the computer, which I rarely do, and went to bed.
In the morning, everything worked.

I couldn't help but remember *Tom Knight and the Lisp Machine*:

> A novice was trying to fix a broken Lisp machine by turning the power off and on.
>
> Knight, seeing what the student was doing, spoke sternly: “You cannot fix a machine by just power-cycling it with no understanding of what is going wrong.”
>
> Knight turned the machine off and on.
>
> The machine worked.
