---
layout: post
title: git hooks, Ctags & Rails
---

Small adjustment to Tim Pope's outstanding [Effortless Ctags with Git](http://tbaggery.com/2011/08/08/effortless-ctags-with-git.html):

To get ctags to also index all bundled gems (including Rails), make the following change to the `ctags` hook

Replace

{% highlight bash %}
git ls-files | \
  ctags --tag-relative -L - -f"$dir/$$.tags" --languages=-javascript,sql
{% endhighlight %}

with

{% highlight bash %}
ctags --recurse --tag-relative -f"$dir/$$.tags" --languages=-javascript,sql `bundle show --paths` `git ls-files`
{% endhighlight %}

After triggering the hook, the `tags` file takes a few seconds to generate in the background - have patience :)
