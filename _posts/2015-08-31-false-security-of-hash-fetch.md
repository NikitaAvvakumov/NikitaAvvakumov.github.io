---
layout: post
title: Don't let Hash#fetch give you a false sense of security
---

On the face of it, Hash's `fetch` instance method is a pretty handy and fail-safe way of
retrieving keys from a hash. It allows one to specify a default value to be
returned if a key does not exist.

{% highlight ruby %}
hash = { :foo => 'bar' }
hash.fetch(:foo, 'default')   # => 'bar'
hash.fetch(:baz, 'default')   # => 'default'
{% endhighlight %}

<!--more-->

That's pretty useful behaviour. However, not infrequently I find myself having
to work with hashes in which a key may be missing *or* be explicitly set to
`nil`. (If you have any control over the situation, just don't create an ambiguous data
structure like that. If not, read on.) In cases such as this, `fetch` is no
help: the key is present in the hash, and the method dutifully retrieves its
value. In fact, I find that using `fetch` has tripped me up more than once
because I rashly assumed that I was handling the missing value while I was
really only handling the missing key.

Consider:

{% highlight ruby %}
book1 = { :title => "100 Years of Solitude", :author => "marquez" }
book1.fetch(:author, "Author unknown").capitalize   #=> "Marquez"

book2 = { :title => "Autumn of the Patriarch" }
book2.fetch(:author, "Author unknown").capitalize   #=> "Author unknown"

book3 = { :title => "No One Writes to the Colonel", :author => nil }
book3.fetch(:author, "Author unknown").capitalize
# => NoMethodError: undefined method `capitalize' for nil:NilClass
{% endhighlight %}

So, as convenient as `fetch` seems at first, it can still leave the application
not only with unsightful `nil`s but with downright unacceptable exceptions.

By contrast, `(book3[:author] || "Author unknown").capitalize` will work with
both missing keys and nil values, and is at least as performant as `fetch`.

`Hash#fetch` remains a good tool to use with well formed hashes, particularly because
it can take a block that specifies more complex default behaviour, as in

{% highlight ruby %}
book3.fetch(:author) do
  @books_with_missing_author += 1
  "Author unknown"
end
{% endhighlight %}

However, one has to be aware of the method's limitations, as well as of the
nuances of the data structures being parsed.
