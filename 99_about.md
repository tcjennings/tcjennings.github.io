---
layout: page
title: About The Intranet of Stuff
permalink: /about
---
# The Intranet of Stuff

Wikipedia has an article that describes the [Internet of Things][] :

>The internet of things (IoT) is the network of physical devices, vehicles, buildings and other items—embedded with electronics, software, sensors, and network connectivity that enables these objects to collect and exchange data.

The IoT describes a mesh of ubiquitous connectivity and telemetry. At a base level, it's what makes things "smart." We can have conversations about "Smart appliances" and even "Smart Homes", but conversations are happening around "Smart Cities" now. The scale is ever-expanding and the IoT has its fingers in a lot of pies.

I want to scale it back down. I want to describe my own universe and make it "Smart." So it's the *Intranet of Stuff*. It's my phone, my appliance, my home. Ultimately, it's my life, and I'm going to see how much of it I can waste making the stuff around me "Smart."

# Making the Dumb Smart

Home automation is almost completely mainstream now; you can find tools to make dumb things smart in just about every home improvement super store, Best Buy, and now Target. Some of the biggest and smallest companies are putting out automation products and frameworks, including Apple's Homekit, Amazon's Alexa, Samsung's SmartThings, and of course the maker of the gold standard of automated lighting, Philips.

The same ubiquitous high-speed Internet access that gives us Netflix, Hulu, and other reasons to cut the cord is bolting cloud logic to our homes through services like If This Then That and Workflow. We're doubling-down on our smart phones and democratizing a field that used to be the domain of million-dollar [Crestron] installations.

Of course, that all means a million different protocols bolted together with shims and hacks and dependence on a service provider for basic functionality.

No wonder most of the stuff you see on display are about as interesting as the Clapper. "Turn your lights red when your favorite stock price goes down!" and "Unlock your front door with your smart phone!" aren't exactly use cases that get me excited about smart home tech. But I want it anyway.

If all I was going to do was turn my porch light on at dusk and off at dawn, I wouldn't bother writing about it. Sure, that's a useful algorithm to have running, but it's the Hello World of home automation. I want to accomplish some things with a little more bite to 'em, which will start with a write-up as a [Use Case][].

# Comment Policy

Right now I have no intention of adding comment functionality to posts or pages. If you want to discuss some content, see the Contact Policy. You could create a thread in a public forum and bring my attention to it.

# Contact Policy

If you want to contact me, send me an @ message on Twitter or an email at the listed address.

You can also visit or follow my [Reddit profile][].

# Advertising Policy

No ads. If I link to a product and there's an opportunity to embed an affiliate ID, I will do so. 

# Posts About The Intranet of Stuff

{% for post in site.tags.about %}
<ul>
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date">{{ post.date | date: "%B %-d, %Y"  }}</span>
  </li>
</ul>
{% endfor %}

[Internet of Things]: https://en.wikipedia.org/wiki/Internet_of_things
[Crestron]: https://www.crestron.com
[Use Case]: /use_cases/
[Reddit profile]: https://www.reddit.com/user/IntranetOfStuff/
