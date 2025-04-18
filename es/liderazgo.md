---
layout: default
title: Liderazgo
permalink: /es/leadership/
lang: es
---

## Liderazgo y Mindset

El liderazgo es un viaje continuo de aprendizaje y adaptación. Aquí comparto reflexiones sobre la importancia del mindset, la resiliencia y el crecimiento personal en el ámbito profesional.

### Publicaciones

{% assign posts = site.categories["Liderazgo"] %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url }}) <small>({{ post.date | date: "%d-%m-%Y" }})</small>
{% endfor %}
