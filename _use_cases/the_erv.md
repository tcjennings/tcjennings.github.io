---
title: The ERV
layout: page
---
While most homes have an HVAC system, not every home has an [ERV][]. According to Wikipedia,

>Energy recovery ventilation (ERV) is the energy recovery process of exchanging the energy contained in normally exhausted building or space air and using it to treat (precondition) the incoming outdoor ventilation air in residential and commercial HVAC systems.

What it means is that stale air in the house is moved outdoors, and fresh air is brought in. Ideally, energy is moved from one stream to the other (incoming heat is exhausted in the summer, and outgoing heat is preserved during the winter).

The ERV in my home is installed in a "partially dedicated" manner. Traditional HVAC systems collect "return air" from the home space and move it to the forced air unit. My ERV is "partially dedicated" because it collects air from specific points in the house and exhausts it, and introduces an equal amount of air to the HVAC return.

My home is ducted such that the bathrooms each have a dedicated return that feeds into the ERV. There are three bathrooms, one of which has a separate Throne Room, so there are four dedicated returns to the ERV. The rest of the space is collected by the ordinary HVAC return.

One consequence in this design is that my bathrooms don't have traditional ventilation fans; the ERV is meant to perform this duty. Another is that in practice the ERV doesn't do a very good job of dehumidification of incoming air, so when it runs during the height of the humid midwestern summer, it tends to dump a lot of new, damp, air into the house.

For these and other reasons, my goal is to make the ERV "smart". It should run when it's a good idea for it to run, and not when it's a bad idea for it to run.

Some ideas in this area include:

* When someone takes a shower, run the ERV to exhaust the humid bathroom air.
* When it is pleasant outside, run the ERV *n* minutes of every hour to keep the inside air fresh.
* When indoor air becomes too stale, run the ERV to refresh it.
* When it is unpleasant outside, do not run the ERV.

The ERV itself is pretty dumb. Out of the box, it's either on or off. The builder installed a basic wall switch in the utility room where it's installed, but this is a simple on-off switch and you have to go down into the room to access it. Other, fancier, after-market wall controls exist, but these have the same proximity problem: you have to go to where the wall switch is in order to control the ERV.

<img style="float: right;" src="/images/erv-lv-block.png" width="200 px" />

Luckily the ERV has a low-voltage wiring block I can use to my advantage. By creating a connection between two terminals I can turn it on; by creating a connection between two other terminals, I can switch it to high speed.

Specifically, I can use dry contact relays to make these connections:

* Jumper between ON and LOW initiates low speed operation.
* Jumper between ON and HI initiates high speed operation.

I can make do with that. Some of the fancier wall controls have as many as 5 speed settings, but for my purposes switching between Low and High seems fine.

In a purely mechanical world, I could use a 3-position switch to make these connections, where the ON terminal is the common line and I switch it between LOW and HI (with the third position being OFF). But I do not want a mechanical solution, I want a smart solution.

# Posts About the ERV

{% for post in site.tags.erv %}
<ul>
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date">{{ post.date | date: "%B %-d, %Y"  }}</span>
  </li>
</ul>
{% endfor %}


[ERV]: https://en.wikipedia.org/wiki/Energy_recovery_ventilation
