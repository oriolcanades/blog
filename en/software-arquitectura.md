---
layout: default
title: Software Architecture
permalink: /en/software-architecture/
lang: en
---

## Software Architecture

Software architecture is fundamental to the success of any project. Here I share my thoughts on architectural patterns, system design, and best practices.

### Posts

{% assign posts = site.categories["Software Architecture"] %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url }}) <small>({{ post.date | date: "%d-%m-%Y" }})</small>
{% endfor %}
