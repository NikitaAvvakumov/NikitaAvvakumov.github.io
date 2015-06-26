---
layout: post
title: block_given?
---

A method of the Kernel module, and thus available everywhere in Ruby, `block_given?` allows you to check whether a block was passed as an argument to a function.

{% highlight ruby %}
def new_func
  if block_given?
    yield
  else
    'No block!'
  end
end

new_func    
# => "No block!"
new_func { 'In the block!' }
# => "In the block!"
{% endhighlight %}

Here's a less contrived example, which came out of my solution to a problem on [RubyMonk](http://rubymonk.com/learning/books/1-ruby-primer/problems/156-sum):

{% highlight ruby %}
class MyArray
  attr_reader :array

  def initialize(array)
    @array = array
  end

  def sum(initial_value = 0)
    # your code here
    @array.inject(initial_value){ |sum, n| sum += (block_given? ? yield(n) : n) }
  end
end

my_array = MyArray.new([1, 2, 3])
my_array.sum(10)
# => 16 (10 + 1 + 2 + 3)
my_array.sum(0) {|n| n ** 2 }
# => 14 (1**2 + 2**2 + 3**2)
{% endhighlight %}

Without a block argument, the method simply adds array elements to a specified initial value. When a block is passed, each array element is first yielded to the block (which squares the element in the example) and *then* summed.