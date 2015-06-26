---
layout: post
title: 'do..end vs {}: Not just a matter of style!'
date: 2014-04-04 11:54:40.000000000 +03:00
tags:
- block
- do..end
- Ruby on Rails
- syntax
---

"Use curly braces for single-line blocks and `do..end` for multi-lines blocks." After seeing various permutations of that statement dozens of times, one can come to believe that the difference between the two is purely stylistic and that there is no practical reason to opt for one vs. the other. Most of the time, that is, indeed, true. Until that one-time-in-a-thousand when it's not.

The trouble with `do..end` vs. `{}` can show up thanks to (a) Ruby's laid-back attitude towards syntax, and (b) the fact that the two kinds of block delimiters do not behave 100% identically.

The syntax slackness in question is the optional usage of parentheses around method parameters. For instance, the following line pairs of code are identical:

{% highlight erb %}
FactoryGirl.create(:user) # is the same as
FactoryGirl.create :user

<%= link_to('Sign in', signin_path) %> # is the same as
<%= link_to 'Sign in', signin_path %>
{% endhighlight %}

The parentheses can be safely omitted from method calls in the vast majority of cases. That is, unless a block is involved, which brings us to (b).
The not-so-subtle difference between `do..end` and `{}` is their order of precedence - the curly braces rank higher on that score than `do..end`. This means that in a parentheses-less method call, the curly braces will bind the block to the preceding parameter, while `do..end` will bind it to the method. 

Example:
{% highlight bash %}
$ rails console
2.1.0 :001 > puts [1, 2, 3].map { |x| x * 2 }
2
4
6
2.1.0 :002 > puts [1, 2, 3].map do |x| x * 2; end
#<Enumerator:hex_code>
{% endhighlight %}

The first command was equivalent to
{% highlight bash %}
puts([1, 2, 3].map { |x| x * 2 })
{% endhighlight %}

while the second was interpreted as
{% highlight bash %}
puts([1, 2, 3].map) { |x| x * 2 }
{% endhighlight %}

In the first case, the block was passed to `.map`, which evaluated it and then gave the result back to `puts`. In the second case, the block was passed to `puts`, which ignored it and only reported that calling `.map` on an array yields an `Enumerator` object.
