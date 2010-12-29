---
layout: post
title: A DSL implementation based on Bundler
---

Here is an implementation of a real basic DSL. The implementation was lifted
from [bundler](https://github.com/carlhuda/bundler/blob/1-0-stable/lib/bundler/dsl.rb).

### Summary

The main idea is you have a class `Dsl` that you call `instance_eval` on. Your
domain specific language is still written in ruby, but by evaluating it within
the context of an instance of `Dsl`, you can create the illusion of top level
language constructs. These are merely public instance methods of `Dsl`. This is
the way bundler evaluates a `Gemfile`.

### Example

Here is an example, using the implementation in the gist below.

{% highlight ruby %}
  dsl = Remote::Dsl.new
  dsl.server :app1, "admin@192.168.1.1"
  dsl.server :app2, "admin@192.168.1.2"
{% endhighlight %}

By using `instance_eval` we can turn the above into

{% highlight ruby %}
  server :app1, "admin@192.168.1.1"
  server :app2, "admin@192.168.1.2"
{% endhighlight %}

Examples of this type of DSL:

* Gemfile
* Capfile
* Rakefile
* config/routes.rb in Rails

### Implementation

<script src="https://gist.github.com/757889.js?file=bundler-style-dsl.rb"></script>