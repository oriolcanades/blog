---
layout: default
title: Arquitectura de Software
permalink: /es/software-architecture/
lang: es
---

## Arquitectura de Software

La arquitectura de software es fundamental para el éxito de cualquier proyecto. Aquí comparto mis reflexiones sobre patrones arquitectónicos, diseño de sistemas y mejores prácticas.

### Publicaciones

{% assign posts = site.categories["Arquitectura de Software"] %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url }}) <small>({{ post.date | date: "%d-%m-%Y" }})</small>
{% endfor %}
