---
layout: post
title: Enjoy your day-of-the-week :)
date: 2014-03-13 10:27:21.000000000 +02:00
tags:
- Ruby on Rails
- time and date
---

Simple:

{% highlight bash %}
$ rails console
2.1.0 :001 > Time.now.wday
  => 4   #Thursday
2.1.0 :002 > Date::DAYNAMES
  => ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]
2.1.0 :003 > "Enjoy your #{Date::DAYNAMES[Time.now.wday]}."
  => "Enjoy your Thursday."
{% endhighlight %}