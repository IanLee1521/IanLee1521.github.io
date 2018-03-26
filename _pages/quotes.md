---
title: Favorite Quotations
permalink: /quotes/
layout: single
classes: wide
author_profile: true
---

{% for quote in site.data.quotes %}

> {{ quote.quote }}
>
> -- {{ quote.name }}{% if quote.date %} - {{ quote.date }} {% endif %}

{% endfor %}
