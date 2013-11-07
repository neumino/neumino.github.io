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
{% endfor %}
