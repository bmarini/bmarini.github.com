---
layout: post
title: Rails 3 modularity
published: false
---

I was reading through `ActionDispatch::Routing::Mapper` to understand how the
routing DSL was implemented and ended up learning a really nice technique for
modularizing a class. Here is the gist of it:

<script src="https://gist.github.com/759204.js?file=modular-classes.rb"> </script>

Probably not something you want to do all the time. For framework code, where
you want to provide building blocks that people can pull out bits and pieces
of functionality, or swap out pieces with their own, this is pretty much how
rails 3 does it.