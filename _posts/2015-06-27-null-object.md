---
layout: post
title: Empowering Null Objects in a Rails application
---

Null Object is a well-known software design pattern that allows one to take advantage of
duck-typing encouraged by Ruby. Plenty of articles offer good descriptions of
this pattern and provide basic usage examples, which has prompted me to attempt
integrating it into my Rails apps several times. Unfortunately, null objects
always seemed to fall short of being able to fully replace nil checks and other
conditionals. Luckily, Sandi Metz took a terriffic in-depth look at this pattern
in a [recent talk](http://confreaks.tv/videos/railsconf2015-nothing-is-something).
Inspired by this presentation, I returned to the Null Object pattern and finally
implemented it in a way that unlocks its full potential while leaving the code
base clean and compartmentalized.

<!--more-->

## Implementing a rudimentary Null Object in a Rails application

A common example of using null objects in a Rails app has to do with
`current_user`. With `user.id` stored in the session, a simple helper method might be
implemented as

{% highlight ruby %}
def current_user
  @current_user ||= User.find_by(id: session[:user_id])
end
{% endhighlight %}

As written, the method returns a `User` object if a user is signed in or `nil`
otherwise. This necessitates a nil check in the view:

{% highlight erb %}
<% if current_user %>
  Welcome, <%= current_user.name %>
<% else %>
  Welcome, Guest
<% end %>
{% endhighlight %}

A basic Null Object implementation can go a short way towards mitigating the
problem:

{% highlight ruby %}
# app/models/guest.rb
class Guest
  def name
    "Guest"
  end
end
{% endhighlight %}

{% highlight ruby %}
# helper
def current_user
  @current_user ||= (User.find_by(id: session[:user_id]) || Guest.new)
end
{% endhighlight %}

{% highlight erb %}
<%# view %>
Welcome, <%= current_user.name %>
{% endhighlight %}

This implementation is not only messy but also half-hearted, as it merely
shuffles the conditional out of the view and into the helper. One way to clean
it up might be to override the `find` and/or `find_by` methods of `User`:

{% highlight ruby %}
# app/models/user.rb
class User
  def self.find_by(args)
    super || Guest.new
  end

  def self.find(args)
    begin
      super
    rescue
      Guest.new
    end
  end
end

# the helper returns to its original implementation
def current_user
  @current_user ||= User.find_by(id: session[:user_id])
end
{% endhighlight %}

While less than ideal, this implementation of the Null Object pattern is
functional, and this is where many article examples stop.

## Going beyond the basics

There are some questions that can help take this implementation of the Null
Object pattern further:

- Should the `User`
class, which already handles persistence of user records and possibly other
duties, be responsible for creating and returning new `Guest` objects? (Class
naming is deliberate here: had the null object been named `NullUser`, it would have been
more tempting to have `User` return a "null version of self". A `Guest`, on the
other hand, is clearly a distinct entity from `User`.) Or does this responsibility
belong to a view helper, whose job is just to provide some reusable convenience methods
for views? Neither option feels right.

- What happens when one wishes to extend the view in this fairly common way:

{% highlight erb %}
Welcome, <%= link_to current_user.name, current_user %>
{% endhighlight %}

Rails is smart enough to route a reference to a `User` object to
`UsersController#show`, but it won't be able to do anything with an instance of
`Guest`. Not yet, anyway.

## Extracting new responsibility
To address the first question, one can ask if determining whether a person is a
user or a guest is a job worthy of a brand new class. Sandi's talk makes a
strong case for answering this in the affirmative. In her presentation, she
drives the point home by naming the new class `GuaranteedAnimal` - whether the
sought `Animal` object exists or not, the `GuaranteedAnimal` class will return
something that will look and waddle and quack like an `Animal`.

And so, the `User` class can lose its custom `find` and `find_by` methods and resume its
(hopefully) single responsibility of persisting user data. Instead, a new class is
created to encapsulate the `User`/`Guest` dichotomy:

{% highlight ruby %}
class Person
  def self.find(id)
    User.find_by(id: id) || Guest.new
  end
end

# helper
def current_user
  @current_user ||= Person.find(session[:user_id])
end
{% endhighlight %}

(It makes sense to propagate the `user` -> `person` change to the
helper method and into the views, but I'm keeping it as `current_user` for
clarity.)

Just like `GuaranteedAnimal`, `Person.find` is now sure to return an object that
will respond to all messages sent to it by other components of the application.
This approach eliminates the need for type checks and neatly encapsulates the
responsibility for determining the type of person interacting with the app.

## Empowering the null object
What about being able to do
{% highlight erb %}
<%= link_to current_user.name, current_user %>
{% endhighlight %}

Rails knows how to route to a `User` because **routes.rb** probably contains
{% highlight ruby %}
resources :users
{% endhighlight %}

Soooooo... Why not
{% highlight ruby %}
resources :guests
{% endhighlight %}

If the application is made with any care, there should never be a reason to
handle anything but `show` for a guest, so `resources :guests` is overkill,
but it drives the point home - treating this null object as a first-class
citizen in the app allows it to perform all the duties of a `User` doppelganger.
`GuestsController` may do nothing more than redirect to sign-up/sign-in
pages, but it serves the higher purpose of putting `Guest` on fully equal footing with `User`, which
allows the former to stand in for the latter in all usage scenarios.

## Conclusion

Null objects exist to eliminate the need for nil checking (which is just a
variant of type checking), and to fully enable Ruby's duck typing and
polymorphism. However, null objects can be easily crippled when they are
pigeonholed into the "support" category, and their half-baked implementations
can force their code to pollute other classes. Code awkwardness and SRP
violations can be cleaned up by diligently extracting new objects to encapsulate
discrete bits of functionality. Moreover, full potential of null objects can be unlocked
only when they are treated as first-class citizens within an application.
