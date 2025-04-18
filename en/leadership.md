---
layout: default
title: Leadership
permalink: /en/leadership/
lang: en
---

## Leadership and Mindset

Leadership is a continuous journey of learning and adaptation. Here I share reflections on the importance of mindset, resilience, and personal growth in the professional realm.

### Posts

{% assign posts = site.categories["Leadership"] %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url }}) <small>({{ post.date | date: "%d-%m-%Y" }})</small>
{% endfor %}
