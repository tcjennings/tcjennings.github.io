---
layout: page
title: Netatmo Personal Weather Station
---
* TOC
{:toc}

Coming soon.

Check out [Netatmo](https://www.netatmo.com/en-US/product/weather-station).

# Pages About NetAtmo

<ul>
{% assign sorted = site.netatmo | sort: 'moddate' | reverse %}
{% for item in sorted %}
  <li>
    <a href="{{ item.url }}">{{ item.title }}</a>
    <span class="date">{{ item.moddate | date: "%B %-d, %Y"  }}</span>
  </li>
{% endfor %}
</ul>

