---
layout: post
title: Rails 3 routing DSL implementation
---

I picked out the bare essentials from the rails 3 routing code to expose how
this DSL is implemented. Here it is.

<script src="https://gist.github.com/757889.js?file=rails-routes.rb"> </script>

`RouteSet`'s responsibility is to store the routes. It hands off responsibility
for creating those routes to `Mapper`.

`Mapper` contains the DSL, the high level API layered onto `RouteSet`.