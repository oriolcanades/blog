---
layout: default
title: Digital Identity
permalink: /en/digital-identity/
lang: en
---

## Digital Identity

Exploring topics such as eIDAS2, OID4VC, self-sovereign identity, trust protocols, and decentralized models.

### Posts

{% assign posts = site.categories["Digital Identity"] %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url }}) <small>({{ post.date | date: "%m-%d-%Y" }})</small>
{% endfor %}

