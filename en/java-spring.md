---
layout: default
title: Java Spring
permalink: /en/java-spring/
lang: en
---

## Java Spring Development

Java and Spring are powerful tools for enterprise application development. Here you will find resources, reflections, and best practices to make the most of these technologies.

### Posts

{% assign posts = site.categories["Java & Spring"] %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url }}) <small>({{ post.date | date: "%d-%m-%Y" }})</small>
{% endfor %}
