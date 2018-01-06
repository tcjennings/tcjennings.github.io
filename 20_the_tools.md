---
layout: page
title: The Tools
---

The tools involved in making the dumb smart aren't all high-tech wizardry, but most of them are. Some tools are real, physical products and others are services. Some are out-of-the-box and some have to be custom-made or customized.

Part how-to, part why-for, and part review, these are the tools I've tried, am using, or are on my wishlist.

{% for thepage in site.the_tools %}
{% if page.title != thepage.title %}
<a href="{{ thepage.url | prepend: site.baseurl }}">
        {{ thepage.title }}
</a>

<p class="post-excerpt">{{ thepage.excerpt | truncate: 160 }}</p>

{% endif %}
{% endfor %}      
