---
layout: default
title: Blog
---

{% include JB/setup %}

{% for post in site.posts %}
<div class="post">
<i class="icon-post"></i>

    <h2><a href="{{post.url}}" class="title">{{post.title}}</a>
    <span class="meta_info">posted on {{post.date | date: "%d %B %Y"}}</span>
    </h2>
    {{post.content}}
</div>

{% if forloop.first %}
<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- Blog - 60 - BnW -->
<ins class="adsbygoogle"
style="display:inline-block;width:468px;height:60px"
data-ad-client="ca-pub-0353610365188957"
data-ad-slot="8201832207"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
{% endif %}
{% endfor %}
