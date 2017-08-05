---
layout: default
permalink: /posts/
---

<ul class="posts">
{% for page in site.posts %}
  <li class="post">
    {{ page.date | date: '%B %d, %Y' }} -- <a href="{{ page.url | relative_url }}">{{ page.title | smartify }}</a>
  </li>
{% endfor %}
</ul><!-- posts -->
