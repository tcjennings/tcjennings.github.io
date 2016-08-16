---
title: Installing GE Z-Wave Smart Dimmer Switches
layout: post
tags: lighting smartthings
---

*Disclaimer: This post is not meant as advice or tutorial for performing electrical work. Treat it only as food for thought when considering your own electrical projects. Every house is different: there is no guarantee what color your wires will be or how exactly your switches are wired.*

I haven't yet put any of my lighting on smart switches. I do have a few [Hue][] products set up as accent lighting, and I have a couple of dry-contact relay switches hooked up to my [ERV][] but so far the rest of my switches are as the builders left them; dumb up-down toggle switches.

My downstairs open-concept rec-room is split into four different entertainment zones: a wet bar, a home theatre, a seating lounge, and a dry bar. There are 5 sets of switches controlling lighting through this area:

<ol type="A">
<li>A single recessed light at the foot of the stairs, 3-way switched with a toggle at the top of the stairs.</li>
<li>Wet bar pendant lighting, which is not yet installed, controlled by a single switch.</li>
<li>6 recessed lights that cover the bar, theatre, and seating area. This is a 4-way switch.</li>
<li>A switched outlet at the dry bar, which is low-output accent lighting.</li>
<li>A pair of recessed lights in the hallway that leads to a bathroom and a back bedroom. This is a 3-way switch.</li>
</ol>

The primary goal for controlling the lighting down here is to manage the recessed lights at `A`, `C`, and `E` using [GE Z-Wave Smart Dimmer Switches](http://amzn.to/2aVk9nE). Of course, it's complicated since they're all 3-way switches (one of them being a 4-way) installed into 5 boxes:

1. At the top of the stairs, a single-gang switch for `A`.
2. At the bottom of the stairs, a double-gang switch for `A` and `C`.
3. At the wet bar, a double-gang switch for `B` and `C`.
4. At the close end of the hallway, a double-gang switch for `C` and `E`.
5. At the far end of the hallway, a single-gang switch for `E`.

The one good thing about all this is that it's all new up-to-code wiring installed in 2015. There are neutrals at each switch box, as is required by code and for smart switches, and I suspect the multi-way switches to be pretty straightforward to install.

# Not Your Ordinary 3-Way Switch

An ordinary 3-way switch works like a normal switch with the addition of something called a "traveller" wire which *travels* from the main switch to the second switch. The way it works is that the 120 volts of "line" juice is always present at both switches, but one switch or the other completes a circuit to the "load" (the light). This means the traveler has two conductors, one or the other of which will be "hot" depending on the disposition of the switches. It's not the easiest concept to wrap one's head around, and it doesn't help that there are [at least nine different ways](http://www.easy-do-it-yourself-home-improvements.com/3-way-switch-wiring-diagram.html) to wire the thing up.

A smart switch works completely differently. There's only one switch involved in switching the line voltage to the load, and the auxiliary switches *never touch line voltage*. Instead, the [auxiliary switches](http://amzn.to/2ba04rl) use the traveler wire for communication back to the main switch. For this reason, your line and load conductors need to be in the same box for the master switch, and all you need in the other box for the aux switch is the traveller and a neutral.

At least in the case of the GE Smart switches, you can add a number of aux switches to accomplish 3-, 4-, or 5-way switching. The travellers from each one just get connected *all together* and plugged into the master switch's single "traveller" jack.

Before I pull stuff out of the walls and start reconfiguring things, I used my [project board]({% post_url 2016-08-01-project-board %}) to preview the setup -- it allows me to make sure the new switches are operational and join them to my Z-Wave network before permanently installing them. This also gives me a "known good" configuration so that if it doesn't work after I install them for real, I know it's probably a wiring issue, not a configuration problem.

# It Takes An Algorithm

A regular dumb mechanical switch does one thing: connect or disconnect the line from the load. A "hot" line from the circuit breaker comes into the box, and is connected to one  terminal on the switch. The line going to the load is attached to the other terminal, and everyone's happy. In the back of the box, the neutral from the load is connected to the neutral from the line and the switch never touches it.

In a standard 3-way switch, the "hot" *line* from the circuit breaker enters one box and connects to the "common" terminal of a switch; a pair of conductors are connected to an additional two terminals on this switch -- the travellers. The travellers run over to the second switch box and connect to the two traveller terminals on the second switch. Finally, the *load* line connects to the "common" terminal of the second switch and goes on to the light or whatever.

To decide how to install a smart switch to replace a standard 3-way switch, the first question is "In which box do I install it?" There are two boxes, two switches to replace, but only one of the new switches is "smart" -- only one will participate in Z-Wave and repeat messages, etc. The other is a fairly brainless auxiliary switch.

I considered:

* Prefer a single-gang box over a double-gang box.
* Smart switches participate in the Z-Wave network by extending the mesh and repeating command signals, so don't cluster them too close together.

All things being equal, you can pick one box or the other. Consider the case of the hallway lights, which I described above as lighting group `E`: a single-gang switch at one end of the hallway and a double-gang at the other. I decide to put the smart switch in the single-gang box and the auxiliary in the double.

The wiring looks like this:

![image]

It turns out in my case that I've decided to place the smart switch in the "load" box. Since the traveller wires work differently for smart switches, the order of operations to install the thing goes something like this:

1. Identify the line and the load. Using a multimeter, I figured out which box had the "line" and which box had the "load". This is fairly easy, since the box with the "load" will have 0V at its "common" terminal when the light is off, and 120V when it is on; the "line" box will have 120V at the "common" terminal in both cases.
2. Turn off the circuit breaker for *every* switch installed in a box in which I'll be working -- if neighboring switches in a 2-gang box are on different circuits, disabling one still means you might brush your knuckles against the other and get a shock.
3. Disconnect all wires and remove the switch from the "line" box. In the diagram, this is the left-hand box. The colors might not match, so you have to make sure to keep track of what comes off "common" and what are the travellers.
4. Connect the "line" directly to one of the travellers. I use a [2-port push-on connector](http://amzn.to/2aWufV6) to accomplish this[^1]. This creates continuity from the circuit breaker to the other box. It makes sense to mate color-to-color here, say black-to-black.
5. Disconnect all wires and remove the switch from the "load" box. This is the right-hand box in the diagram. See step 3.
6. The GE Smart Switch has 5 terminals: line, load, neutral, ground, and traveller. Hook up the ground first, then the neutral. The switch comes with a short length of "jumper" cable to use with the neutral; connect one end to the bundle of white neutrals in the box and the other to the neutral terminal on the switch.
7. Connect the "load". The conductor I removed from the "common" terminal on the old switch goes to "load" on the new smart switch.
8. Connect the "line". One of the traveller wires is the "line" since I connected them together in step 4. This might be the black wire.
9. Connect the "traveller". The other traveller wires will connect to the auxiliary switch in the other box. This might be the red wire.
10. Back at the other box, hook up the aux switch, which has only 3 terminals: ground, neutral, and traveller. Neither line nor load is connected to this add-on switch. Connect ground and neutral as in step 6.
11. Connect the traveller wire (maybe red) to the traveller terminal.
12. Turn on the circuit breaker.

Because I prototyped the switch on a project board, it's already joined to my network and ready to go. If I wired it up correctly, I should be able to immediately control the lights via Z-Wave, but a manual test is the first thing to try. The smart switches look like Decorator paddle switches, but even in 3-way installations up is always "on" and down is always "off."

When I am successful, I can continue my project by installing more switches in the other locations using the algorithm above. In the 5 boxes, I ended up installing:

1. At the top of the stairs, a smart switch for `A`.
2. At the bottom of the stairs, an aux switch for `A` and a smart switch for `C`.
3. At the wet bar, a dumb switch[^2] for `B` and an aux switch for `C`.
4. At the close end of the hallway, an aux switch for each of `C` and `E`.
5. At the far end of the hallway, a smart switch for `E`.

So: I haven't installed two smart switches in the same box, instead have kept them separate to improve the coverage of my Z-Wave network. This also slightly simplifies the wiring since double-gang boxes are a bit more crowded.

In a future post, I'll talk about the aftermath and using these devices in [SmartThings][].

[Hue]: /the_tools/hue
[SmartThings]: /the_tools/smartthings
[ERV]: /use_cases/the_erv
[image]: http://22wmo83kfu4a2go2xhiedyzgkk.wpengine.netdna-cdn.com/wp-content/uploads/files/pictures/hk.jpg

[^1]: I don't know why these things aren't standard-issue. They're a lot easier to use, especially for homeowners who don't spend 8-12 hours a day twisting 14-gauge wire together, and even more especially in tight spaces. All you need is 0.5" of *straight* stripped wire, and the connector does the work for you. They're UL-listed, which means they aren't "cheating", and they even take up less space in the box than a twist-cap.
[^2]: Since the bar pendant lighting (`B`) isn't yet installed I just replaced the dumb toggle switch with a dumb Decora switch so the two switches in this box are the same shape and I don't have to get a goofy hybrid faceplate.