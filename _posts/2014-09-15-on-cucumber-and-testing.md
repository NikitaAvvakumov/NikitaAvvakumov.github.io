---
layout: post
title: On Cucumber and testing
---

_This article was originally written as part of an ongoing discussion within the
team building the [OSRA](https://github.com/AgileVentures/osra) app._

## 1. Cucumber.

Any time a component is added to an application, its anticipated benefits must
be weighed against the costs of its implementation. Cucumber is a costly
framework in terms of development time, developer effort, and cognitive strain
on both creators and readers of tests due to the following:

- Gherkin is an awkward sort-of-but-not-quite English
- Scenarios written in Gherkin need to be then translated into Ruby in a
  separate file (both good and bad, see below)
- Regex in step definitions are laborious to write and may require periodic
  maintenance (good for learning regex, bad for productivity)
- Maintenance & correct reuse of step definitions can be overwhelming in larger
  test suites
- Cucumber features run much slower than specs

Thus, for its use to be justified, Cucumber has to give returns that are
somehow greater than the investment put into writing its tests. I am not
convinced of Cucumber’s utility as documentation, nor of its usefulness as a
shared language for developers. Developers speak Ruby, or are in the process of
learning to speak it, and so Ruby, rather than Gherkin, is our _lingua franca_.

When it comes to documentation, I tend to question the value of any such thing
that is external to the code base itself. Developers can read code in models
and controllers. They can also read specs, and therefore have no need for an
additional framework. From the perspective of the client or user, Cucumber
stories are hardly suitable as manuals. Imagine reading them in order to learn
how to use GitHub or Heroku. Moreover, for the kinds of apps we usually build,
a need for a manual or any other kind of external document would be strongly
indicative of design and architectural flaws. My conclusion, therefore, is that
we need to evaluate Cucumber’s merits strictly as a testing framework. Its
other contributions to a project are secondary or non-existent.

## 2. Testing at the right level of abstraction.

Feature tests are meant to test an application at the highest level of its
functionality. Here, there are no models, no requests, no Rails - just a user’s
interaction with the browser. When we feature-test e.g. creation of a new
user... Wait, that's low-level language... When we feature-test new user sign
up, what is relevant? My answer is something along the lines of:

{% highlight gherkin %}
When I sign up for Site Name Then I should see my new
user profile And I should receive a confirmation email
{% endhighlight %}

i.e. We are testing that sign-up works. Step definition take care of filling in
requisite fields and clicking on relevant buttons.  But all we want to know -
all we ask the scenario to test - is whether the sign-up functionality works.

I find the following example from the
[shoulda-matchers](https://github.com/thoughtbot/shoulda-matchers/blob/master/features/rails_integration.feature#L4)
gem inspiring:

{% highlight gherkin %}
When I generate a new Rails application
{% endhighlight %}

This step’s definition has 16 lines of code, including a conditional, but the
step itself is clean and uncluttered with unnecessary detail. The only bit of
information relevant to this high-level test is that a new Rails app is
created, the “how” of it is not.

Later, the scenario has these steps:

{% highlight gherkin %}
Then the output should contain "2 examples, 0 failures"
And the output should contain "should require name to be set"
And the output should contain "should respond with 200"
{% endhighlight %}

I would argue that this descends to too-low a level of abstraction. To stay
consistent with the initial paradigm, these specifics ought to be used as
definition for a single step

{% highlight gherkin %}
Then the specs should pass
{% endhighlight %}

How they pass, what examples they run, or what messages they return is not the
concern of the high-level feature. Interestingly, if Cucumber is used thus, it
immediately becomes less costly:

- scenarios are short, to the point, easier to follow and to correct;
- only a few awkward-English steps need to be translated to Ruby;
- all the browser interactions are written in our beloved Ruby (Capybara) right
  from the start;
- the need for regex and paths is drastically reduced if not outright
  eliminated.

This happens because the Cucumber framework very intentionally promotes such
high levels of abstraction by “physically” separating test steps from their
implementation. (RSpec features can be made as abstract, but with extra effort - 
otherwise, all the `click_button`s & `fill_in`s need to be included right in
  the scenario.)

Admittedly, my experience developing web apps and testing them is rather
limited at the moment. You’d be right to question my opinions. If it helps, the
very people who develop Cucumber happen to [think
likewise](https://github.com/cucumber/cucumber-rails/issues/174). web_steps.rb
was [removed from the
framework](http://aslakhellesoy.com/post/11055981222/the-training-wheels-came-off)
precisely so that people would stop writing bloated, boring and brittle
scenarios that contain the low-level steps like `When I click on …` or `Then I
should see ... within ...`.

## My happy place

Let us keep Cucumber. It’s a good tool. It can be useful to us and to the
client. And if it doesn't come in terribly handy in building OSRA, getting
experience with it will still be good for us. But let us use Cucumber well and
in the way it was built to be used by keeping our tests at the highest level of
abstraction. The low-level details have two possible places to go: step
definitions or view specs. Both of these can co-exist without redundancy:
thoughtbot uses features for high-level tests, and view specs to test
conditional display of elements in view templates, and has good reasons for
doing so.
