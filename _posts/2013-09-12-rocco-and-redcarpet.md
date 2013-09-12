---
layout: post
category : Geek
tags : [ruby, rocco, redcarpet]
title: Roco and Redcarpet
---
{% include JB/setup %}

I recently ran into an issue with the [rocco gem](http://rtomayko.github.io/rocco/)
using [redcarpet](https://github.com/vmg/redcarpet). The error was:

```bash
/var/lib/gems/1.9.1/gems/rocco-0.8.2/lib/rocco.rb:447:in `process_markdown`: uninitialized constant Rocco::Markdown (NameError)
```

One way to fix it is to replace line 36 of `/var/lib/gems/1.9.1/gems/rocco-0.8.2/lib/rocco.rb` with

```rb
libs = %w[redcarpet/compat rdiscount bluecloth]
```

Basically redcarpet was updated and rocco did not change its code to match the new redcarpet.
