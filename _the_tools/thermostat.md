---
layout: page
title: Smart Thermostat
---
* TOC
{:toc}

Coming soon.

# Pages About Smart Thermostat

<ul>
{% assign sorted = site.thermostat | sort: 'moddate' | reverse %}
{% for item in sorted %}
  <li>
    <a href="{{ item.url }}">{{ item.title }}</a>
    <span class="date">{{ item.moddate | date: "%B %-d, %Y"  }}</span>
  </li>
{% endfor %}
</ul>

