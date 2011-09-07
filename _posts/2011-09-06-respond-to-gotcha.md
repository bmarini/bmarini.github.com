---
layout: post
title: Rails respond_to gotcha
---

Ran into this problem during the launch of a new site. We has some innocent
looking controller code that looked something like this:

{% highlight ruby %}
  class SitesController < ApplicationController
    respond_to :json, :html

    def index
      expires_in 1.day, :public => true
      @sites = Site.all

      respond_to do |format|
        format.json { render :json => @sites }
        format.html
      end
    end
  end
{% endhighlight %}

This looks fine, except that a certain version of IE 6 will pass an
`Accept` header of `*/*` to URLs that have no extension, such as `http://example.com/sites`.

That means Rails will serve of the first format you've defined in your `respond_to`
block, in this case the `json` version of `sites#index`.

One might say, who really cares if a handful of IE 6 users is getting a browser
full of `json` instead of my site? I'm not even supporting IE 6 anymore!

A valid argument, except, for this:

{% highlight ruby %}
  expires_in 1.day, :public => true
{% endhighlight %}

If you are using a cache such as Memcached, Varnish, or even Akamai in
aconfiguration that respects the `Cache-Control` headers from your Rails App,
one IE 6 user could cache your `http://example.com/sites` URL as `json`
for 24 hours (in this case). Meaning all other users hitting that URL will get
`json` back instead of `html` until the cache expires.

The easy fix:

{% highlight ruby %}
  class SitesController < ApplicationController
    respond_to :json, :html

    def index
      expires_in 1.day, :public => true
      @sites = Site.all

      respond_to do |format|
        format.html
        format.json { render :json => @sites }
      end
    end
  end
{% endhighlight %}

Make sure `html` is the first format in the `respond_to` block.