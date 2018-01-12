---
layout: page
title: The House
---
My house was built in 2015 to LEED standards. I think it earned LEED Silver certification. A lot of what that means was baked into the construction process and doesn't have a daily impact on my life, but there is a particular appliance involved that is central to managing the air quality within the home. The lighting is exclusively LED (except for a lamp or three with legacy CFLs in them), and the appliances are all Energy Star-certified, as far as I know. The HVAC is a 95% efficient heat pump+gas furnace. The windows are all engineered with UV-filtering materials to keep heat out and save my stuff from fading; the exception is a large south-facing architectural detail window that earned some LEED points with its passive-solar effect.

I put not-quite-enough copper into the walls to support most networking tasks, but that's largely independent of home automation gear, which tends to be wireless anyway. There's lots of places to plug in a PC or a game console, hook a TV up to antenna or cable, or plug in a pair of speakers.

There's a security system with your typical array of magnetic reed sensors for external doors, motion detectors, fire and smoke detectors, and keypads. They're all wired in and separate to the kind of home automation I'm going to implement, but it would be nice to have a notification or status bridge between the security panel and its devices and the home automation platform. After all, if I want to do something when a door opens, I don't want redundant sensors on that door to figure it out.

{% for thepage in site.the_house %}

{% if page.title != thepage.title %}
<a href="{{ thepage.url | prepend: site.baseurl }}">
        {{ thepage.title }}
</a>

<p class="post-excerpt">{{ thepage.excerpt | truncate: 160 }}</p>

{% endif %}
{% endfor %}      

