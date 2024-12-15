---
layout: page
title: Categorías
permalink: /categories/
---

Explorar artículos por categoría:

{% for category in site.categories %}
## {{ category[0] }}

<ul>
  {% for post in category[1] %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a> ({{ post.date | date: "%d-%m-%Y" }})</li>
  {% endfor %}
</ul>
{% endfor %}
