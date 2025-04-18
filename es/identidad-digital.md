---
layout: default
title: Identidad Digital
permalink: /es/digital-identity/
lang: es
---

## Identidad Digital

Exploro temas como eIDAS2, OID4VC, identidad soberana, protocolos de confianza y modelos descentralizados.

### Publicaciones

{% assign posts = site.categories["Identidad Digital"] %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url }}) <small>({{ post.date | date: "%d-%m-%Y" }})</small>
{% endfor %}
