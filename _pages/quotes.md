---
title: Favorite Quotations
permalink: /quotes/
---

{% for quote in site.data.quotes %}

> {{ quote.quote }}
>
> -- {{ quote.name }}{% if quote.date %} - {{ quote.date }} {% endif %}

{% endfor %}
