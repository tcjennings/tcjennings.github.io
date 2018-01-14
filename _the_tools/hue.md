---
layout: page
title: Hue Smart Lighting
---
* TOC
{:toc}

Coming soon.

Check out [Hue](http://www2.meethue.com/en-us/).

# Pages about Hue

<ul>
{% assign sorted = site.hue | sort: 'moddate' | reverse %}
{% for item in sorted %}
  <li>
    <a href="{{ item.url }}">{{ item.title }}</a>
    <span class="date">{{ item.moddate | date: "%B %-d, %Y"  }}</span>
  </li>
{% endfor %}
</ul>

