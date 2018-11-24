---
title: Talks
permalink: /talks/
---

{% assign talks = site.data.talks | sort: 'date' | reverse %}
{% for talk in talks %}

## {{ talk.title }}

<script async class="speakerdeck-embed" data-id="{{ talk.speakerdeck_id }}" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

{% endfor %}
