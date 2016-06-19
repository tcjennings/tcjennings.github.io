---
layout: page
title: The Tools
---


{% for page in site.the_tools %}


<a href="{{ page.url | prepend: site.baseurl }}">
        {{ page.title }}
</a>

<p class="post-excerpt">{{ page.excerpt | truncate: 160 }}</p>

{% endfor %}      
