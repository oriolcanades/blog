---
layout: page
title: Publicaciones
permalink: /posts/
---

<ul>
{% for post in site.posts %}
    <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <p><small>Publicado el {{ post.date | date: "%d de %B de %Y" }}</small></p>
    </li>
{% endfor %}
</ul>