---
title: Chris Douglas
subtitle: Blog Posts
layout: defaults/page
permalink: index.html
fluid: true
---

{% for post in site.posts limit:5 %}
{% include components/post-card.html %}
{% endfor %}


