---
title: SmartThings Multipurpose Sensor
layout: post
tags: smartthings
---

[SmartThings](/the_tools/smartthings) was nice enough to send me a pair of their [Multipurpose Sensors](https://shop.smartthings.com/#!/products/samsung-smartthings-multipurpose-sensor), which I've been meaning to pick up.

These little guys pack 4 different sensors into a small form factor (which while small is not discreet), but you can only really use 3 of them at a time[^1].

[^1]: Technically, it really is just three sensors, but one of these is a 3-axis accelerometer, the output of which is "vibration" and "tilt", which are 2 different sorts of measurements, the former in the X-Y plane and the latter involving the Z-axis.

# The Device

At about 2" by 1.5", the sensor itself is small; the same form factor as other SmartThings sensors, like the discontinued [Temp/Humidity Sensor][] that this sensor presumably replaces. It's about 0.75" deep when attached to the optional wall-mount bracket, so like most battery-powered sensors it sits like a wart on whatever surface to which you attach it, which given its utility is probably your door trim.

The senor is powered by a single 3V CR-2450 battery. Other SmartThings sensors I've used in the same form factor with the same battery have lasted over a year so far, so I might expect similar longevity from this unit.

Like the other first-party SmartThings sensors, this guy runs on the ZigBee network instead of Z-Wave, which means it doesn't take advantage of the network repeating ability of various Z-Wave devices around the house. To test its useful range, I've installed one of these sensors in my office, which is the front bedroom of the house. The house is largely square-ish, but this room juts out the front such that 3 of its four walls are exterior walls. This positioning makes the room particularly challenging for wireless technology. Both the SmartThings hub and my Wi-Fi AP are in the more-or-less centered utility room, and Wi-Fi has trouble covering very deep into this room.

I don't expect the ZigBee signal to have much trouble. I have some ZigBee [outdoor lighting][] whose receiver is similarly placed, but outdoors, and it's been working fine on its Sunrise-Sunset schedule.

# The Sensors

This unit is primarily an open/close sensor, and as such it has two parts: the big white blister with the battery in it and a smaller blister with a magnet in it. When the two parts are near, the magnet in the small blister closes a switch in the big blister and the state of the system changes from "open" to "closed". This is how all open/close sensors work, including the wired, recessed ones I have drilled into my exterior doors and connected to my alarm panel.

Rounding out the capabilities of the device are a temperature sensor and a vibration sensor.

I'd like to have a humidity sensor in this pack, too, since the other first-party humidity sensor is, as I mentioned, discontinued, and I find humidity a more compelling measurement for my home. Well, wish in one hand...

The fourth sensor in the unit is an "orientation" or "tilt" sensor, which can be used *instead* of the reed sensor to report open-closed status, e.g., on a garage door where vertical orientation is "closed", and horizontal is "open". This is why I say you can really only use 3 of the 4 sensors at a time, at least in the stock first-party Device definition. There may be third-party or open-source device apps for this unit which expose the accelerometer as a true discrete sensor.

To use the accelerometer instead of the reed sensor, there's a setting in the app to turn on "Garage Door Mode", which does not require the secondary magnetic unit but does require that the main blister be aligned in a particular way, which I cannot find detailed in the packed-in quick-start guide. The blister has a recessed line along one of its edges, which is the "Magnet Alignment Mark", and the unit must be positioned such that this line is to the right. When placed correctly, the sensor reports "closed" when in the vertical position.

I'm not sure what's up with the other axes of the accelerometer, which presumably make up the "vibration" sensor. Installed on my door trim, the little panel in the SmartThings app that reports vibration (it looks like a treasure map for some reason) always reports "Active." This doesn't seem likely, as it's just sitting there. This may be a defect as it only manifests in one of the two sensors.

# The Applications

As a "Multi" sensor, these units lend themselves to a number of applications, some of which are obvious:

* Take some action when a door is opened or closed.
* Track usage of an appliance through vibration, like a washing machine or dryer. I've seen someone online use one of these on a string inside his air return to determine when the HVAC blower was active (there are obviously better approaches).
* Take some action when a temperature threshold is met or unmet.
* Detect tampering with some object through vibration, like a window pane or some box of valuables.

That covers some of the "What"s. Using these drivers, here are a few "Why"s:

* Turn on [ventilation][] when a bathroom door is closed.
* Perform some kind of notification when the laundry cycle ends.
* Trigger an [alarm][] when tampering is detected in "Away" mode.
* Use temperature readings to make smart [thermostat][] decisions.

Those are some pretty obvious ideas. What I'm trying to consider is "Why do I care if a door is closed or not?" I mentioned that my exterior doors already have reed switch sensors tied to my alarm panel, so those are covered. I'm left with my interior doors and a two-part question:

* Do I care if this door is opened or closed?
* What am I going to do about it?

If the answer to the second question is "Nothing," then obviously the answer to the first question should have been "No."

Let's see what I've got:

* Is the door to the freezer in the garage closed? If not, perform some kind of notification.
* Like above, but with "wine cellar in the pantry".
* When a bathroom or water closet door is closed, turn on high-speed ventilation.
* If a window is opened, suppress the air conditioning and/or prevent putting the alarm in "Away" mode.
* Did the bar cabinet open when I'm on vacation and I told the house-sitter to stay out of my booze?

Generally, I don't care if a bedroom door is opened or closed (people with kids certainly might), but I might care about the local temperature of a bedroom for informational or climate control reasons. I would have to weigh the cost (including battery life) of a purpose-built temperature (and hopefully humidity) sensor against the cost of this multi-sensor knowing that I probably wouldn't depend on the other sensors in some applications -- you might never use the vibration sensor in a door application.

# The Wishlist

When it comes to "multi" sensors, that "multi" can mean a lot of things. I already mentioned wanting a humidity sensor in this thing, but one thing I haven't seen in *any* multi-sensor is a glass-break detector. Maybe it's impractical, I don't know, but a sensor like this one is asking to be installed on a door or window, and the location you choose leads to compromises.

* Installed on a door, the vibration sensor is less useful, but the temperature sensor is probably more representative of the room's comfort.
* Installed on a window, the vibration sensor is useful, but the temperature sensor is probably biased from drafts or direct sunlight.

You might think in that second case that a vibration sensor on a window is a good security measurement, but it will depend greatly on how sensitive the accelerometer is. A good breeze or a clap of thunder can certainly cause some vibration at a window, as can a bird or large insect collision (the collision doesn't need to be with the bird itself, as anyone who's had to clean windows may know). Inside the house, a cat can climb to a comfortable perch on a sill and cause vibration. There are lots of potential false alarms in reading just "vibration" so I would not be comfortable using one of these sensors to determine if someone was trying to break into my house. A glass break sensor is much more suited to this use case, but it's not present, so nevermind. Yet I believe I have convinced myself that the second bullet above is not really the case: the vibration sensor may not be useful on a window -- it depends entirely on how sensitive the accelerometer is, and since my sensor is stuck on "active" around-the-clock, it's either way too sensitive or flat-out busted.

# Next Steps

I'm going to use these sensors for a week or two in a particular application: one on my office door, and the other on the washing machine. I'll collect the stats from the sensors and see how they stack up against my expectations in reporting interval and accuracy. After that, I'll look into any third-party or community device drivers and see how far under the hood I can really peak, or whether I'm stuck with simple boolean results for the accelerometer.

[Temp/Humidity Sensor]: {% post_url 2016-07-18-smartsense-temp-humidity-sensor %}
[outdoor lighting]: http://amzn.to/2fxtTYO
[ventilation]: /use_cases/the_erv.html
[alarm]: /use_cases/the_alarm.html
[thermostat]: /the_tools/thermostat.html
