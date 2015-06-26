---
layout: post
title: Using Date - Ruby vs. Rails
---

This little investigation was prompted by an interesting quirk whereby `Date.today == Date.tomorrow` returned `true`.

`Date.today` is implemented in Ruby and uses system’s local time & timezone. 

In Rails, timezone can be set in **config/application.rb** but defaults to UTC. Thus, it can be different that local system time used by Ruby.

`Date.tomorrow` & `Date.yesterday` are implemented in Rails and use `Date.current` internally.

`Date.current` is implemented in Rails thus:

{% highlight ruby %}
::Time.zone ? ::Time.zone.today : ::Date.today
{% endhighlight %}

i.e. if `Time.zone` is set (which in Rails it is, as stated above), `Date.current`, `Date.tomorrow` & `Date.yesterday` will use `Time.zone.today`. If it’s not set, which never happens in Rails, then the methods fall back to the timezone-unaware Ruby implementation. 

The solution to avoiding issues related to all this in Rails is to use the Rails implementations based on `Date.current` and avoid using Ruby’s `Date.today`.