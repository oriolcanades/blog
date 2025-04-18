---
layout: default
title: Posts
permalink: /en/posts/
lang: en
---

<ul>
{% for post in site.posts %}
    <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <p><small>Published on {{ post.date | date: "%B %d, %Y" }}</small></p>
    </li>
{% endfor %}
</ul>
