---
layout: post
title: Nuking all changes with Git
date: 2014-03-13 15:19:44.000000000 +02:00
tags:
- Git
- git-clean
- git-diff
- git-reset
- revert
---

After looking it up twice in one (less than productive) day...

{% highlight bash %}
$ git reset --hard # delete all changes from tracked files
$ git clean -f -d # remove untracked files and directories
{% endhighlight %}

*(Warning: This is destructive! Code will be irreversibly lost.)*

An alternative to doing the above would be:

{% highlight bash %}
$ git add .
$ git reset --hard
{% endhighlight %}

And if things aren't a complete disaster that deserves to be annihilated - for example, perhaps they can serve as a stern reminder in the future - they can be saved prior to destruction with

{% highlight bash %}
$ git diff > ~/lessons_learned/rails_mess_1.diff
{% endhighlight %}
