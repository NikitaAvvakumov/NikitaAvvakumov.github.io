---
layout: post
title: RSpec 3.0.1 and RubyMine
date: 2014-06-18 08:15:28.000000000 +03:00
tags:
- RSpec
- Ruby on Rails
- RubyMine
---

After the recent RSpec update, warnings started showing up in RubyMine:

> Deprecation Warnings: Treating `metadata[:execution_result]` as a hash is deprecated. Use the attributes methods to access the data instead. Called from .../teamcity/rspec3_formatter.rb: line#

(So, RSpec is complaining that RubyMine's formatter is not accessing its data properly?) The warnings were overwhelming to the point of obscuring the results of running the test. Simple fix:

1. open **/Applications/RubyMine.app/rb/testing/patch/bdd/teamcity/spec/runner/formatter/teamcity/rspec3_formatter.rb** by clicking on one of the links in the warning message

2. this takes you right to the offending lines (#135, 333 & 340 in my version), wherein, as per the message's instructions, the hash format

{% highlight ruby %}
started_at_ms = get_time_in_ms(example.execution_result[:started_at])
{% endhighlight %}

needs to be changed to an attribute accessor

{% highlight ruby %}
started_at_ms = get_time_in_ms(example.execution_result.started_at)
{% endhighlight %}

No more warnings :)

(In the process, I also learned that while **File > Open Recent** works to reopen *projects*, **View > Recent Files** (or `⌘-E`) is for recently closed *files*.)
