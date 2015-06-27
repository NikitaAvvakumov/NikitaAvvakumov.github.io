---
layout: post
title: Selective stubbing of method calls in RSpec
---

While building a simple API in Rails, I had the following method in my User model that needed to be tested:

{% highlight ruby %}
def generate_auth_token!
  loop do
    self.auth_token = Devise.friendly_token
    break unless self.class.exists?(auth_token: auth_token)
  end
end
{% endhighlight %}

The method uses Devise's `friendly_token` method to generate an authentication token for a new user record. After calling the method once, the loop checks whether a user record with the same auth token already exists. If the answer is positive, the method goes for a second try; otherwise it breaks out of the loop, mission accomplished.
<!--more-->

To test this, then, it is necessary to simulate the situation where the first call to `friendly_token` returns an auth token already taken by another record, while a subsequent call does not. Enter the block implementation of RSpec Mocks.

{% highlight ruby linenos %}
  describe '#generate_auth_token!' do
    it 'generates a new token when one is taken' do
      original_friendly_token = Devise.method(:friendly_token)
      call_count = 0
      allow(Devise).to receive(:friendly_token) do
        if (call_count += 1) == 1
          'veryunique123'
        else
          original_friendly_token.call
        end
      end

      existing_user = FactoryGirl.create :user, auth_token: 'veryunique123'
      user.generate_auth_token!
      expect(user.auth_token).not_to eq 'veryunique123'
    end
  end
{% endhighlight %}

On L3, the original implementation of the method being stubbed is assigned to a variable. Within the block, call count (1 or > 1) determines whether the method is stubbed to return a predetermined value or whether the response is delegated to the method's original implementation. (To avoid even the slightest possibility of a randomly failing test, I could have stubbed the subsequent calls to return a distinct preset value instead of hoping that `Devise.friendly_token` will never return `"veryunique123"`.)

Obviously, the block's conditions can be customized to fit the test's needs, and the entire block implementation can be used for other purposes as well, as you can find out <a href="http://www.relishapp.com/rspec/rspec-mocks/v/3-2/docs/configuring-responses/block-implementation#delegate-to-partial-double's-original-implementation-within-the-block">in the RSpec docs</a>.
