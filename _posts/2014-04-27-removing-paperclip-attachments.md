---
layout: post
title: Removing Paperclip attachments
date: 2014-04-27 13:54:08.000000000 +03:00
tags:
- deleting attachments
- Paperclip
- Ruby on Rails
---

Paperclip's [GitHub page](https://github.com/thoughtbot/paperclip#deleting-an-attachment) includes the necessary steps to delete an attachment - the model's attribute that refers to the attachment simply gets set to nil. Here is the full implementation of deleting a user's profile picture in a Rails app. Start with tests (right?!):

**spec/features/editing_users_spec.rb:**
{% highlight ruby %}
require 'spec_helper'
feature 'Editing Users' do
  let(:user) { FactoryGirl.create(:user) }
  let(:admin) { FactoryGirl.create(:admin) }
  scenario 'removing user profile photo' do
    sign_in_as admin # of course
    visit edit_user_path(user)
    click_link 'Remove profile photo'
    expect(page).to have_content 'User profile photo has been removed.'
    expect(page).not_to have_css("img[src*='user.png']") # file name of the user photo defined in the factory
    expect(page).to have_css("img[src*='paperclip_default.png']") # specified in the User model
  end
end
{% endhighlight %}

([This post]({% post_url 2014-04-27-paperclip-attachments-in-factories %}) covers creation of an attachment in a FactoryGirl factory.)

<!--more-->

Red. The test first fails when it tries to find a "Remove profile photo" link. In my implementation, photo removal takes place in the Edit view, so that's where the link goes (or, to be more precise, into the 'form' partial):

**app/views/users/_form.html.erb:**
{% highlight erb %}
<%= form_for @user do |f| %>
  <%= f.label :photo, 'Profile photo' %>
  <%= f.file_field :photo %>
  <% if @user.persisted? %>
    <%= link_to 'Remove profile photo', remove_user_photo_path %>
  <% end %>
.
.
<% end %>
{% endhighlight %}

(If the user has not been persisted to the database yet, as is the case when the partial is used by the New view to create a new user, the `link_to` cannot be generated as there is no user id yet - see **routes.rb** below. This is why it is wrapped in the `if..persisted?` clause.)

Red. In this iteration, the test fails because `remove_user_photo_path` is undefined. That's next:

**config/routes.rb:**
{% highlight ruby %}
Rails.application.routes.draw do
  .
  .
  get 'user/:id/remove_photo', to: 'users#remove_photo', as: 'remove_user_photo'
end
{% endhighlight %}

As can be seen from the route, the `link_to` method used in the view will generate a url such as `/user/1/remove_photo`, which is why having a user id (from a persisted user) is necessary.

Red. The test now fails because the `remove_photo` action does not exist in the Users controller. We are finally ready to make use of Paperclip's instructions on deleting attachments:

**app/controllers/users_controller.rb:**
{% highlight ruby %}
class UsersController < ApplicationController
  .
  .
  def remove_photo
    @user = User.find(params[:id])
    @user.photo = nil
    @user.save
    redirect_to @user, flash: { success: 'User profile photo has been removed.' }
  end
end
{% endhighlight %}

The action pulls the relevant user record out of the database, sets its attachment-referring `photo` attribute to `nil`, and saves the user to the database. It then redirects the browser to the user's profile page, where the profile image now displays Paperclip's default graphic and not the photo defined in the factory.

Green :)
