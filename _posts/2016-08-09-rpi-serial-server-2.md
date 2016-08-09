---
title: Raspberry Pi Serial Server Part 2
layout: post
tags: the_stereo smartthings
---

In [Part 1][] I detailed attaching a [Raspberry Pi][] to my [Monoprice 6-zone amp][] via a serial cable and getting them talking with the help of a [Node][]-based [app][]. I left off after suggesting that my end goal was to integrate the Amp into my [SmartThings][] ecosystem using this Rpi<->Serial<->Amp hookup.

# Why Integrate?

The Monoprice multi-zone system works pretty well all by itself, mostly because I had the opportunity to wire this [house][] myself. While I didn't get to run every piece of copper I wanted to run, I did get some category-5 cables run to locations where I could install the control keypads for 5 of the 6 zones available on the unit:

* In the kitchen.
* Between the living room and dining room (open concept; these "rooms" are relative)
* In the master bath suite.
* At the wet bar.
* In the studio.

The last zone I was undecided about at the time, so I didn't run any control cable for its keypad. The dry bar? Another bathroom? The guest bedroom? You never really run out of places you might want to call an "Audio Zone" but you do run out of speaker cable and the time to install it. While the amp is expandable with two more chassis to make a total of 18 potential zones, I don't think I'll bother with more than 6.

I have, in the intervening time, decided what to do with the last zone, though: Outdoors. Originally I figured I could make the deck and patio speakers a part of Kitchen zone since there's *proximity* to its keypad. Since the Monoprice amp has both power and line-level outputs for each zone, I wired it up such that the Deck and Patio speaker runs went through local impedence-matching volume controls to a dedicated amp (which in turn is powered on or off depending on the 12V signal from the Kitchen zone). I fed the Kitchen zone's power outputs to its speakers and the line-level outputs to this dedicated amp.

Little did I know at the time, but it turns out that the "line level" outputs for each zone are actually pegged to the *volume level* of the zone's amp. I think this is a bizarre-bordering-on-stupid design decision, since it means even though the two outputs are available they're not independent in any way.

By swapping a couple of cables (the line out cables and the 12V trigger cable), I can move the outdoor speakers to their own zone, but the problem then is I don't have a keypad with which to control it. This is one good reason to integrate the amp into a larger control system.

Another couple of reasons to integrate this device is that it takes too long (5 seconds) to turn off a zone at the keypad, and "mute" functionality is missing from the keypad's physical controls.

For being so low-priced and from an unlikely source (for a long time I avoided Monoprice's active electronics, but I've clearly turned the corner on that opinion), the Monoprice multi-zone amp is an integrator's dream come true. It features RS232 serial control, PA override, 12V triggers for each zone, IR emitter channels for each zone, and as mentioned is expandable to 18 total zones.

# Why SmartThings?

The answer to this is simple. Once the decision to integrate is made, it makes sense to make it part of the main ecosystem driving the "smarts" of the rest of the house, which happens to be SmartThings, for the reasons detailed on the [page I wrote about it][].

The work I did in Part 1 of this series is probably sufficient to integrate the device into a stand-alone dashboard, but it lacks the necessary plumbing for SmartThings. Using the Node app detailed previously, I can send commands to the unit over the serial interface and get back responses in JSON. That's probably sufficient for something like [Dashing][], or I could write some simple HTML front-end to it and put a shortcut on my phone, and it would be plenty impressive and operational.

SmartThings, on the other hand, has the potential to make the amp participate in other activities and (love or hate the cloud) do so from anywhere. I won't belabor the point. I have a Raspberry Pi with a serial connection to a device, and I have a SmartThings hub. What I would like next includes:

* Basic zone control from my phone, since at least one of my zones doesn't have a keypad.
* Mute a zone from my phone, since I can't do that from a keypad.
* Power off all zones at once, instead of walking to every room and doing the 5-second button-mash.
* Change source channel for a zone from my phone.
* Change volume from my phone.

# How, Then?

This is all basic integration stuff, and it's all stuff the RS232 control language supports, so I'm not too worried about that. The question is "How do I do it?"

I know SmartThings is an ecosystem of Device Handlers, which talk to hardware and SmartApps, which perform tasks with those Devices, but I've never implemented a Device from scratch.

Turns out, I won't have to. Not today, anyway.

As luck would have it, someone already did most of the hard work for me. A SmartThings community member, let's call him [@redloro][], wrote a stack of software to integrate his [Russound][] multi-zone amplifier with SmartThings, and he did it in a way that leaves it extensible and useful to people who might have similar use cases for different hardware.

The stack he wrote includes:

* SmartThings Node Proxy, a Node application that knows how to talk to SmartThings on one side and to arbitrary devices on the other.
* A Russound RNET plugin for the above, which communicates with hardware over a serial port and emits JSON in response.
* A Russound Zone Device Handler, which is a SmartThings app that defines how the platform should communicate with the Node Proxy for a given Zone.
* A Russound Multi-zone Amp SmartApp, which glues it all together: it talks to the Node Proxy to figure out what Zones are available, then spawns Zone devices to cover them. It also "subscribes" to updates from the Node Proxy and parcels out responses to the Zone Devices as necessary.

It's pretty slick stuff if you have a Russound controller. So my options were to yank out my Monoprice stuff and put in Russound instead, or mod the open source software to my own purposes.

So I did the latter thing, which makes me a contriber to his stack, so now in addition to the above, it includes:

* A Monoprice plugin that knows how to talk to the multi-zone amp over the serial port;
* A Monoprice Zone Device Handler;
* A Monoprice Amp SmartApp.

I have to hand it to @redloro, the SmartThings Device Handler and SmartApp required not much more than a find/replace to change "Russound" to "Monoprice", so almost all of the useful work went into the plugin. It's not often you see people writing custom code that solves for the general case, so it was nice to find this package and be able to extend it seamlessly without changing or breaking anything else.

# Plugged In, Switched On

A plugin is a chunk of code that participates in the framework of some larger piece of software. A Photoshop plugin might extend the base functionality of Photoshop by adding some feature that it lacks, or doing something better than before. There are plugins designed to apply high-quality image filters, or apply specific special effects, or even just replace one color with another in a better way than Adobe wrote.

The SmartThings Node Proxy does two things: talks to SmartThings and hosts plugins. The plugins operate independently of each other and do all the heavy lifting for the back end. The plugin might take a request like `/zones/11/state/` and understand that to mean "What is the power-on state of Zone 11?" Then it translates this into a question it can ask the hardware. In the case of the Monoprice amp, that question is `?11PR`. Not much of a question, but the amp answers `>11PR01` if the zone is powered on and `>11PR00` if it's turned off. The plug-in takes the answer and creates a JSON document that represents it and sends it back to the original requestor.

It sounds simple, and it's not too difficult to accomplish when you have good documentation. The user manual for the Monoprice amp has *adequate* documentation for the RS232 control codes, and the existing plugin for the Russound device made a fine template, so after putting the two together I had a working plugin after a few hours work and testing.

At the end, I sent my work over to him and now the Monoprice plug-in lives comfortably within @redloro's [Github][] repository alongside his other work, where anyone can find and use it for their own projects.

Next time I'll talk about how exactly SmartThings makes use of this thing and some of the things I can do with it.

[Node]: https://nodejs.org/en/
[app]: https://github.com/jnewland/mpr-6zhmaut-api
[SmartThings]: /the_tools/smartthings
[Monoprice 6-zone amp]: http://www.monoprice.com/product?c_id=109&cp_id=10918&cs_id=1091801&p_id=10761&seq=1&format=2
[Part 1]: {% post_url 2016-08-03-rpi-serial-server %}
[Raspberry Pi]: https://www.raspberrypi.org/products/model-b/
[house]: /the_house/
[page I wrote about it]: /the_tools/smartthings
[Dashing]: http://dashing.io
[Github]: https://github.com/redloro/smartthings
[Russound]: https://www.russound.com/products/audio-systems/multi-room-controllers/mca-series-controllers/mca-66-6-zone-6-source-controller-amplifier
[@redloro]: https://community.smartthings.com/users/redloro/activity