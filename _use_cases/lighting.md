---
title: Lighting
layout: page
logline: Lighting is the "Hello World" of smart home automation.
---
# Pages About Lighting

{% for item in site.lighting %}
<ul>
  <li>
    <a href="{{ item.url }}">{{ item.title }}</a>
    <span class="date">{{ item.date | date: "%B %-d, %Y"  }}</span>
  </li>
</ul>
{% endfor %}

# Lighting

Coming soon.

