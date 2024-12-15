---
layout: page
title: Categories
permalink: /categories/
---

# Categorías

Explorar artículos por categoría:

<ul>
{% for category in site.categories %}
  <li><a href=\"/categories/{{ category[0] }}\">{{ category[0] | capitalize }}</a> ({{ category[1].size }})</li>
{% endfor %}
</ul>
