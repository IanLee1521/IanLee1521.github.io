---
title: Talks
permalink: /talks/
---

{% for talk in site.data.talks reversed | sort 'date' %}

## {{ talk.title }}

<script async class="speakerdeck-embed" data-id="{{ talk.id }}" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

{% endfor %}
