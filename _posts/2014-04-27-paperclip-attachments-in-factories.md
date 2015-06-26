---
layout: post
title: Paperclip attachments in RSpec factories
date: 2014-04-27 12:11:46.000000000 +03:00
tags:
- fixtures
- Paperclip
- RSpec
- Ruby on Rails
---

Assuming one has a model whose attachments are handled by Paperclip, i.e.

{% highlight bash %}
$ rails generate paperclip user photo
{% endhighlight %}

**app/models/user.rb:**
{% highlight ruby %}
class User < ActiveRecord::Base
  has_attached_file :photo
end
{% endhighlight %}

then to generate a test model with FactoryGirl, including the attachment:

**spec/factories/user_factory.rb:**
{% highlight ruby %}
include ActionDispatch::TestProcess
FactoryGirl.define do
  factory :user do
    name 'Sample User'
    ...
    photo { fixture_file_upload(Rails.root.join('spec/fixtures/test_img.png'), 'image/png') }
  end
end
{% endhighlight %}

Various manipulations of the attached image through the web interface can then be tested with RSpec and Capybara.
