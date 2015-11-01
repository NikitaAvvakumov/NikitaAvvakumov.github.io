---
layout: post
title: How constant are your constants?
---

When an object should not change during the execution of a program, you make it
a constant:

{% highlight ruby %}
CONST = some_value
{% endhighlight %}

Good forever, right? Well, not really. More accurately, perhaps, not at all.
First, there is the case of a simple reassignment:

{% highlight ruby %}
:001 > CONST = 'a'
 => "a"
 :002 > CONST = 'b'
(irb):2: warning: already initialized constant CONST
(irb):1: warning: previous definition of CONST was here
 => "b"
 :003 > CONST
 => "b"
{% endhighlight %}

Ruby issues a warning, but the constant still gets reassigned. This is perhaps
unfortunate, but at least it's clear what happened. In other cases, the change
of a constant can take place by less obvious means and, to boot, without raising
any warnings.

{% highlight ruby %}
:001 > CONST = [1, 2, 3]
 => [1, 2, 3]
 :002 > CONST.map! { |n| n * 2 }
 => [2, 4, 6]
 :003 > CONST
 => [2, 4, 6]
{% endhighlight %}

What's going on? The critical thing to understand is that constants' intended
use it to keep the object _reference_ unchanged. This is why Ruby issues a
warning when a constant is reassigned to point to another object. However, the
_object_ referenced by the constant is free of any constraint or
supervision from Ruby. Since the bang version of `.map` changes the referenced array in
place, Ruby sees no violation of the constant's "constantness".

The above example is rather contrived, while the change of the constant's value
still happens in an obvious fashion. Here's a version of it that is both more
realistic and less transparent:

{% highlight ruby %}
class Person
  GREETING = 'Hello'

  def initialize(name)
    @name = name
  end

  def greet_politely
    "#{GREETING}, #{@name}"
  end

  def greet_enthusiastically
    greeting = GREETING
    (greeting << ' there').upcase!
    "#{greeting}, #{@name}"
  end
end

joe = Person.new('Joe')
puts joe.greet_enthusiastically

jim = Person.new('Jim')
puts jim.greet_politely

# outputs
"HELLO THERE, Joe"
"HELLO THERE, Jim"
{% endhighlight %}

Jim was supposed to get a polite greeting but got a familiar "HELLO THERE"
instead. This happened because when Joe called `.greet_enthusiastically`, the
variable `greeting` was made to point to the same `"Hello"` string object
referenced by the constant `GREETING`. Via this new reference, the string had
"there" appended to it and then upcased, all in place. The end result of this
was that the class variable was irreversibly modified by a poorly designed
instance method.

The problem of inintentionally changing a constant's value can be mitigated
somewhat by Ruby's object freezing. However, this freezing is shallow and applies to the
object itself but not to its componets. Again, using an array as an example:

{% highlight ruby %}
 :001 > ary = [1, 2]
 => [1, 2]
 :002 > CONST = [ary, 3].freeze
 => [[1, 2], 3]
 :003 > CONST.map! { |n| n * 2 }
RuntimeError: can't modify frozen Array
  from (irb):3:in `map!'
  from (irb):3
  from /Users/user/.rvm/rubies/ruby-2.2.0/bin/irb:11:in `<main>'
 :004 > ary.map! { |n| n * 2 }
 => [2, 4]
 :005 > CONST
 => [[2, 4], 3]
{% endhighlight %}

While the array `[[1, 2], 3]` referenced by the constant could not be modified,
its constituent array referenced by `ary` remained free of any such
constraints. Any of Ruby's methods that modify an object in
place will produce this effect.

_This post was heavily inspired by the description of Ruby constants' behaviour
in [this post](https://bearmetal.eu/theden/how-do-i-know-whether-my-rails-app-is-thread-safe-or-not/)
from Bear Metal._
