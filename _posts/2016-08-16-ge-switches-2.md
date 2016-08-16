---
title: Using GE Z-Wave Smart Dimmer Switches
layout: post
tags: lighting smartthings
---

In a [previous post][] I detailed the installation of a few [GE Z-Wave Smart Dimmer Switches](http://amzn.to/2aVk9nE) in 3- and 4-way installations. In this post I'll detail more of the implementation.

# Inclusion

Inclusion is the process of "including" a Z-Wave device into a network. Usually it involves two steps: put your hub or controller into "discovery" mode, and cause your new device to be discoverable. This isn't very different than pairing a new Bluetooth device, for example.

Because I installed and tested each of my Smart Dimmer Switches on a [project board][] I was able to put them quite close to my [SmartThings](/the_tools/smartthings) hub and do the inclusion before permanently installing them in the walls. This proximity helps inclusion; some devices even suggest performing inclusion within a meter of your hub before installation.

The GE Switches are very simple to include: press the "on" paddle. After asking my SmartThings hub to "find new things" and pressing the paddle, the device showed up as a generic "Z-Wave Dimmer" and I was able to verify operation on my project board and control a pair of LED loads. With that confirmed, I proceeded with the permanent installation.

# Operation

Out of the box, the GE dimmer switches operate such that:

* When turned on, the switch ramps the light up to its last brightness level.
* When turned off, the switch ramps the light down until it is off.
* When a paddle is held, the light is dimmed up or down in 1% steps.

When I say "ramp up" here, I mean the light turns on at 1% and glides evenly through brightness levels until it reaches its target. It does this in steps of 1% and it spends about 30 milliseconds at each step, so "ramp up" from 0% to 99% takes 30 x 100 = 3000 milliseconds (3 seconds). Of course, "ramp down" is the equivalent function in the other direction. This is meant to create a pleasing, clean, gradual sweep of light brightness.

Unfortunately, two facts ruin the scene:

1. Not all LED lights like being dimmed like that, even if they are "dimmable".
2. The default device handler used by SmartThings does not allow you to change the dimming behavior of the switch.

There's not much remedy for `1`. The lights you have are the lights you have. LED lighting is rather notorious for unpredictable dimming performance. An LED is a piece of solid-state electronics that we should be thankful are dimmable at all.  I have LED trim installed in recessed cans all throughout my house, but only one other one is on a dimmer, and it's a rather ordinary analog dimmer that only goes down to maybe 20%.

Consider a rather straightforward LED trim kit like the inexpensive [Commercial Electric][] CER7630 line, which according to specs has a dim range of 10% - 100%. What happens if you try to dim this LED below 10%? Ideally, it would simply turn off at that point, but reality is something different: the LED will flicker or blink like a strobe light, probably at or around full brightness.

So you hook this thing up to a GE Smart Dimmer. It's at full brightness, and you hit the button to turn it off. It starts ramping down through its levels, and it looks great, just like you're drawing the curtains on a bright day. Then it hits that weird threshold, flashes at full output, and goes off.

It's enough to make you think your dimmer is broken. You start experimenting. You set it at, I don't know, 8% brightness and you're in a rave. You kick it to, say, 23% brightness and it's pleasant again.

The moral of the story is that the GE Smart Dimmer is best used with incandescent lighting. It *does* work with LED, but you can't blame it for the shortcomings of your specific LED product. Instead, you need to find a way to work around it.

Unfortunately, unlike some dimmers, the GE Dimmer doesn't have an adjustment dial with which to set the minimum dim level. Instead, it has a couple of [advanced configuration][] settings with which you can play to make it control your lighting in an acceptable manner.

# Nut and Bolts

First things first: to change the configuration of a Z-Wave device is to send specific Z-Wave instructions to that device. As end-users, we're not in the business of crafting or even understanding how to craft a Z-Wave instruction. That's why we have [SmartThings](/the_tools/smartthings) hubs to manage everything. This layer of abstraction, though, means that when we need to do something outside the prescribed abilities of that hub, we just can't do it.

In the SmartThings platform, a Device Handler defines how the hub will communicate with a specific kind of device, and what options and capabilities for that kind of device are exposed to you as a user. As noted above, the GE Z-Wave Dimmer Switch turned up as a "Generic Z-Wave Dimmer" kind of device, which is probably fine for 80% of the users 80% of the time. But it doesn't let me get at those advanced configuration settings I need to keep my lights from triggering seizures every time I turn them off.

SmartThings is an *open* platform, which means when someone finds a gap in the out-of-the-box behavior, they can write a replacement. Given the right set of tools, I can replace "Generic Z-Wave Dimmer" with something more appropriate or specifically designed for an exact device.

The [SmartThings Community](https://community.smartthings.com) is full of tinkerers, developers, and hackers[^1] that do this sort of thing for fun. Since the GE devices are a popular, mainstream piece of kit, I didn't have to look very long or hard to find that someone has already written a replacement Device Handler for this dimmer switch.

Community member [@desertblade](https://community.smartthings.com/users/desertblade/activity) has written a Device Handler named [Enhanced Dimmer Switch](https://github.com/desertblade/Enhanced-Dimmer-Switch) in which he's taken the "Generic" device code provided by SmartThings and added in the GE-specific advanced configuration settings.

To get from here to there, two things need to happen: I need to get his code, and I need to understand how those settings work.

Getting the code is the easy part. Sign in to the SmartThings [IDE][] and hit the "My Device Handlers" tab. I have two options now: I can "Create New Device Handler" and copy/paste his code into a new one, or I can create a link to his Github repository. The former option makes the code entirely my responsibility, and the latter enables me to easily integrate updates that he might make. I take the latter route, the steps for which are documented [elsewhere][], and that new Device Handler is part of my personal SmartThings arsenal.

Still in the IDE, I navigate to the "My Devices" tab and find the listing for my dimmer switch, click through to its detail page, and edit it using the button at the bottom of the page. In edit mode there's a "Type" field where I can select any Device Type from the pool of default devices plus any of my personal devices as defined in "My Device Handlers". Toward the bottom of the list I find the "Enhanced Dimmer Switch" entry and select it; use the "Update" button to commit my choice. I repeat this for any other GE Dimmers I have installed so they're all using the "Enhanced Dimmer Switch" device type.

# Setting the Settings

The Enhanced Dimmer Switch enables a few configuration settings for the device, four of which in particular are useful in affecting the switch's dim rate:

* Z-Wave Dim Steps
* Z-Wave Dim Intervals
* Manual Dim Steps
* Manual Dim Intervals

The first two define how the dim rate should operate when the switch receives a Z-Wave command to dim, as it would when using the SmartThings app or an automation rule to control the dimmer.

The second two define how the dim rate should operate when the switch is manually adjusted, i.e., you stand there and touch it. When you press-and-hold a paddle, it dims the light up or down based on these settings.

As mentioned before, the defaults for these configuration registers is "1" step and "3" intervals. The "step" is the magnitude (in percent) of every change the dimmer will make when dimming up or down. The "interval" is the number of 10-millisecond periods, meaning the light will change "steps" percent every "interval x 10" milliseconds. With those default values, that's a 1% change every 30 milliseconds, so about 3 seconds from maximum-to-minimum or vice versa.

I want to choose settings that will avoid the range of dim levels at which my lights don't cooperate. I can stand there and fool around with the switch, but let's say that below 20% I get undesirable flicker.

Let's also say that I still want it to take 3 seconds to go from maximum-to-minimum.

First, I'll decide that the "Steps" value should be `20`[^2]. Since there are five 20s in 100%, I can say `3000 ms / 5 intervals = 600 ms/interval` and that "600" breaks down into 60 10-millisecond chunks. So I configure steps and intervals for both Z-Wave and Manual control:

* Dim Steps = 20
* Dim Intervals = 60

The interaction of these two values means you can create a lot of different behaviors. Speed it up with 20 steps/40 intervals for a 2-second ramp. Instant on/off would be 99 steps/1 interval[^3].

# That's It

Right. The GE Smart Dimmer Switch. It's mainstream. It's widely available. It's not inexpensive. It does what it says it does. You might have to mess around with SmartThings before you're actually happy with it.

[IDE]: https://graph.api.smartthings.com
[previous post]:{% post_url 2016-08-15-ge-switches %}
[project board]: {% post_url 2016-08-01-project-board %}
[advanced configuration]: http://www.ezzwave.com/advanced-operation/
[Commercial Electric]: http://www.homedepot.com/p/Commercial-Electric-5-in-and-6-in-White-Recessed-LED-Trim-with-2700K-92-CRI-CER6730DWH27/204726945
[elsewhere]: http://docs.smartthings.com/en/latest/tools-and-ide/github-integration.html

[^1]: I use the classical definition of "hacker", which refers to a technology enthusiast with a penchant for tinkering and understanding how things work.
[^2]: This is a kind of imprecise heuristic. If my max value is 99, a step setting of 20 means from 0 I will hit 20, 40, 60, 80, 99 which avoids the trouble spots below 20%. But on the way down we hit 99, 79, 59, 39, 19, 0, and landing on that 19 may or may not cause a visible flicker. You just have to play with the numbers.
[^3]: I will probably come back to this and just set them to instant on/off to avoid any possible voltage issue with my LED drivers. My real goal is to be able to use automations like "When I start Movie Mode, dim the lights to 30%. When I press play, turn the lights out. When I press pause, bring the lights back to 30%." I don't really care how the lights ramp up or down when I use the switch.