---
layout: page
title: Writings
permalink: /writings/
---

{% for post in site.posts %}

### [{{ post.title }}]({{post.url}})

{:.date}

<small>{{post.date | date: '%B %d, %Y' }}</small>

{{post.excerpt}}

{% endfor %}
