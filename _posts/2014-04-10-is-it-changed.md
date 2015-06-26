---
layout: post
title: Is it changed?
date: 2014-04-10 09:35:58.000000000 +03:00
tags:
- ActiveModel
- change
- Ruby on Rails
---

Rails has some useful methods to check whether an object has changed since first being loaded into memory. Here are some examples:

{% highlight bash %}
2.1.0 :001 > user = User.find(1)
 => #<User id: 1, name: "Roderick", email: "rod@example.com", created_at: ..., updated_at: ...>
2.1.0 :002 > user.name = 'Jimmy'
 => "Jimmy"
2.1.0 :003 > user.changed?
 => true
2.1.0 :004 > user.changed
 => ["name"]
2.1.0 :005 > user.changes
 => {"name"=>["Roderick", "Jimmy"]}
2.1.0 :006 > user.name_changed?
 => true
2.1.0 :007 > user.email_changed?
 => false
2.1.0 :008 > user.name_was
 => "Roderick"
2.1.0 :009 > user.save
 => true
2.1.0 :010 > user.changed?
 => false
{% endhighlight %}

More info here: [ActiveModel::Dirty](http://api.rubyonrails.org/classes/ActiveModel/Dirty.html).
