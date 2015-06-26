---
layout: post
title: Learn to delegate (your model methods)
date: 2014-04-17 16:10:42.000000000 +03:00
tags:
- ActiveRecord
- delegate
- Ruby on Rails
---

`delegate` is neat trick that allows one to write cleaner, neater code. Particularly when dealing with ActiveRecord associations, `delegate` makes it possible to "outsource" a method to an associated object.

{% highlight ruby %}
class Book < ActiveRecord::Base
  # has attribute 'author'
end

class Chapter < ActiveRecord::Base
  belongs_to :book
  delegate :author, to: :book
end
{% endhighlight %}

Without `delegate`, when dealing with a `Chapter` object, you'd need to access the book's author via `chapter.book.author`. With delegation, this can be neatly shortened to `chapter.author`, with the `.author` getting handed off to the associated `Book` object.

An object can delegate multiple methods to its association, as shown in the example below. Another handy option is to use *prefixes* with `delegate`. For instance, using the same approach as above to refer to the book's title would create `chapter.title`, which is confusing. With prefixes, however:

{% highlight ruby %}
class Chapter < ActiveRecord::Base
  belongs_to :book
  delegate :author, :title, to: :book, prefix: true
end
 
chapter.book_title   # returns book's title
chapter.book_author  # returns book's author
{% endhighlight %}

(Of course, if chapters had their own titles, the above code would make them inaccessible, but that's what you get with contrived examples. See [this post](http://pivotallabs.com/rails-delegates-are-even-more-useful-than-i-knew/) on the Pivotal Labs' blog for a solution.)

One final tidbit to touch upon is that `delegate` is not limited to deliberately created methods and attributes but extends equally well to those created behind the scenes by the association:

{% highlight ruby %}
class Quote < ActiveRecord::Base
  belongs_to :chapter
  delegate :book, to: :chapter
end

quote.book  # returns the book object associated with the chapter to which the quote belongs
{% endhighlight %}

For more on `delegate`, including custom prefixes and how to handle NoMethodErrors, see the [API Docs](http://apidock.com/rails/Module/delegate).
