---
title: SmartThings SmartSense Temp/Humidity Sensor
layout: page
tags: [smartthings]
moddate: 2016-07-18
---

When I first got my [SmartThings][] hub (version 2), I didn't have much in the way of Things to connect to it. I did have some rough ideas of [use cases][] in mind, so instead of a bunch of switches I went for the low-hanging fruit: battery-powered sensors. Along with that hub I got a [Aeon Labs Multisensor][] and a pair of SmartThings-branded temperature/humidity [sensors][].

[sensors]: https://shop.smartthings.com/#!/products/smartsense-temp-humidity-sensor

As I write this, it appears that this particular sensor is no longer available for purchase, either online or through a retail channel. That's too bad, since it doesn't appear that any other SmartThings-branded sensor includes the humidity capability. The "Multipurpose sensor" that may have taken this guy's place in the product lineup has a lot going for it, but humidity is not among its specs.

This information, then, may be moot. But no matter.

# Installation

The SmartSense Temp/Humidity sensor is a little white rectangle that shares dimensions with the [Water Leak Sensor](http://amzn.to/2aoQK3r), or about 2.5 inches on its long side. It comes with a slightly smaller mounting plate and some 3M mounting tape so you can stick it to a surface or mount it permanently with a screw (which it also comes with, along with a wall anchor).

The SmartThings devices, including the SmartSense Temp/Humidity sensor, are [Zigbee][] devices, so they won't benefit from an existing mesh of [Z-Wave][] devices.

I installed both sensors in my upstairs bathrooms; one in the master above the shower and one in the guest bathroom above the large soaking tub. These are the bathrooms we use to bathe, so they're the rooms most in need of humidity level tracking.

Getting them attached to my hub was easy and each of them paired up without error after reading the tiny booklet that came along with them. As first-party branded devices, I trust that the SmartThings device type they're using is accurate.

# Power and Longevity

Each sensor is powered by a single CR-2 battery, which provides a 3V service to the device. I've had these guys active for 7 or 8 months now and both of them report a battery status of 100%, which seems questionable. Either they truly are amazingly conservative in their battery usage or they simply don't report their status in a way the Device Handler expects.

# Accuracy

It can be hard to gauge the accuracy of a sensor. This device reports both Temperature and Humidity, but without a reference you can't know whether to trust its measurements.

I have a non-smart [Extech Humidity Meter](http://amzn.to/29Q1Uw6) that is expected to be reasonably accurate. If it is not, it has a calibration knob for adjustment. Its accuracy is documented at +/- 5%. I tested this meter by placing it in a humidity chamber[^1] overnight and verifying that it reports humidity values within tolerances.

With this meter in the same location as one of my SmartSense sensors, the sensor and meter report a humidity level within 1% of each other, which I would consider pretty accurate. My [Netatmo Weather Station][], on the other hand, at the same location, reports a humidity level 8-11% higher than this and clearly needs adjustment (curiously, the Netatmo's outdoor module seems more accurate, as it reports a humidity level in line with that of the reported weather).

[^1]: In a sealed container at room temperature, place a good amount of table salt moistened with distilled water until the salt takes on a beach-sand consistency. After a while the humidity in this container will be exactly 75%.

Temperature, too, seems accurate. The SmartSense is within 0.5F of the Extech meter and within 1.5F of my Netatmo indoor module, both of which are within comfortable tolerances compared to my thermostat setting.

# Reporting Frequency

One notable feature of this SmartSense sensor is that it reports measurements to SmartThings pretty infrequently, only when there's a change, and only reports integer values.

This could have something to do with the long battery life I've observed (so far), but it can also be frustrating if you expect or require a sensor with a more reactive sense of duty.

In the following graphic I compare the update frequencies of my two SmartSense sensors with that of my Aeon Labs Multisensor (this comparison only includes temperature and humidity updates; all other measurements from the Aeon Multisensor are excluded). This is a big difference: the SmartSense sensors report in a couple dozen times a day compared to the Aeon sensor's couple hundred.

![Sensor Update Frequency](/images/ss-frh/freq.png)

Since the sensor only appears to update on changes of full-integer values (temperature goes from 73 to 74) instead of fractional changes (from 73.1 to 73.2), in a stable environment you can expect this guy to be relatively taciturn.

# Maybe This Is Why

That all sounds like good news so far. Good battery life, apparently accurate measurements, small footprint -- what's the downside? Well, besides you can't seem to buy the thing anymore, there's a question of prolonged accuracy.

Check out this graph of temperature and humidity over a couple of days this week for these two SmartSense sensors:

![Sensor Accuracy Over Time](/images/ss-frh/compare.png)

The temperature lines in the top graphs aren't too questionable. You can get an idea of the update frequency by looking at the dots plotted along the lines. Each dot is an update; the orange sensor went 20 hours between temperature updates! Even if there were fluctuations in there, you can see how the sensor might be slow to report on them.

The lower graph of humidity measurements is a bit more wonky, especially the yellow sensor line. It has frequent unmotivated upticks of 20%, only to immediately (well, usually immediately) drop back to normal.

The orange humidity line is interesting in that it captures a period of long shower activity which causes a big spike in bathroom humidity which is in turn attenuated by the [ERV][]. This also shows the increase in update frequency when the measurements are rapidly changing (point density during showers is much higher than normal).

The orange sensor also has a couple of unmotivated humidity spikes (i.e., not caused by a shower event, in which case you'd expect the temperature to rise at the same time), but not as many as the yellow sensor.

The best I can conclude from this is that when it works, the SmartSense Temp/Humidity sensor works pretty well, but it is prone to wild swings of humidity measurements, which could cause problems if you plan to implement humidity-based alarms or trigger actions based on that value.

Maybe that's why SmartThings no longer sell a sensor with a humidity measurement.

[Z-Wave]: https://en.wikipedia.org/wiki/Z-Wave
[Zigbee]: https://en.wikipedia.org/wiki/ZigBee
[SmartThings]: /the_tools/smartthings
[use cases]: /use_cases/
[Netatmo Weather Station]: /the_tools/netatmo
[Aeon Labs Multisensor]: http://amzn.to/2a6cuVB
[ERV]: /use_cases/the_erv