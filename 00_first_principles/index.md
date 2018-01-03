---
layout: page
title: First Principles
---

Coming soon.

{% for thepage in site.first_principles %}
{% if page.title != thepage.title %}
<a href="{{ thepage.url | prepend: site.baseurl }}">
        {{ thepage.title }}
</a>

<p class="post-excerpt">{{ thepage.excerpt | truncate: 160 }}</p>

{% endif %}
{% endfor %}      
