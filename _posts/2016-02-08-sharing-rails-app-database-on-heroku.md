---
layout: post
title: Sharing a database between Rails apps on Heroku
---

Heroku has a convenient new way to [share add-ons between apps](https://blog.heroku.com/archives/2015/1/23/expanding_the_power_of_add_ons).
Since their postgres service is one such add-on, it becomes extremely easy to
get two apps to talk to the same database.

<!--more-->

The easiest and most natural way to create the postgres add-on initially is
simply to deploy the first Rails application to Heroku. Other than adding the
`pg` and `rails_12factor` gems to the `production` group of the Gemfile as
usual, no other steps need to be taken here.

The second application can be created and deployed in a similar fashion, and
Heroku will automatically add the postgresql add-on and configure it to point
to a different database. The first step, then, is to immediately remove this
add-on from the second app with `heroku addons:destroy heroku-postgresql`.

Next, you need to find out the name of the first app's postgresql add-on. Run
`heroku addons` from the first app's local folder. The output will contain
something along the lines of `heroku-postgresql (<db name>)`. We are interested
in the `<db name>` part, and it usually comes in the form
`ancient-savannah-12345` as per Heroku's default naming scheme.

Now the second app can be linked to the first app's postgresql add-on by running
`heroku addons:attach <db name> -a <second-app-name> --as <meaningful-name>`.
The `--as <meaningful-name>` part is optional, but it's not a bad idea to give
this shared add-on a name that reflects its role, the fact that it's shared, or
what have you.

Finally, the second app needs to be configured to point to the correct Heroku
database. Near the bottom of `config/database.yml` there are commented-out
instructions for doing just that. Start by removing the default configuration of
the production db, and then uncomment

{% highlight yaml %}
  production:
    url: <%= ENV['DATABASE_URL'] %>
{% endhighlight %}

Since the second app's original postgres add-on was destroyed in the first step,
the `'DATABASE_URL'` environment variable is not set. In the first app's local
folder, run `heroku config:get DATABASE_URL`. The output will look like this:

{% highlight bash %}
$ heroku config:get DATABASE_URL
postgres://XXXXX:YYYYY@#####.compute-1.amazonaws.com:####/#####
{% endhighlight %}

This full string is the address for the first app's postgres server together
with the app's authentication credentials.

Now, in the second app's local folder run `heroku config:set
DATABASE_URL=<postgres server>`. With that done, commit and deploy to Heroku.
To test that the setup is functional, run `heroku pg:psql` from the second app's
folder. You should be able to connect to the shared database and run queries.
