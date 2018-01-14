---
layout: page
title: Samsung SmartThings
---
* TOC
{:toc}

Samsung [SmartThings][] is a home automation platform with a long list of pros and cons. It's not the only gig in town, and it probably isn't the best, but it's well-supported and the barrier to entry is low, even if it does has its problems.

SmartThings started off as an independent platform funded by a Kickstarter campaign in 2012 and was bought by Samsung in 2014, and that company has committed to a 3-comma spend on IoT technology over the next [4 years][], some of which hopefully will go to making the SmartThings platform a real workhorse.

# Some Cons

Right off the bat, a major negative to SmartThings is the fact that it's cloud based. Even though you purchase and host a SmartThings "hub" in your home, and even though Version 2.0 of that hub promised on-premises automation, virtually everything that SmartThings does it does in the cloud. The hub is little more than a box with radios in it.

Another con is the user experience. The UX of the iOS app and the developer portal leaves a lot to be desired. Navigating the app is not intuitive, and important tools seem to be tucked away in inconvenient corners while less-necessary options live front and center. I'm sure this is a lot to do with me, but little of it makes sense.

# Neutral Ground

Not everything about the platform is awful, or else I wouldn't be writing about it. Some aspects I consider "neutral" include:

* Wirelessness. This isn't a fault of SmartThings, but it's the industry in general. Home Automation hubs like SmartThings use a wireless protcol or two to communicate with objects, and a lot of those objects themselves are wireless, which means batteries.
* Given that Samsung is putting SmartThings in mainstream retail environments like Best Buy is a double-edged sword. On one hand, the big face of the platform is simple if-this-then-that automation which doesn't allow for a lot of nuance. The platform is open and completely programmable, so that wins out in the end, but I still have to muscle past the training-wheel screens every time I open the app.

# The Good Stuff

Obviously there has to be a good chunk of positive qualities to make this platform worthwhile.

* Multi-radio support. The hub supports both [Z-Wave][] and [ZigBee][] devices out of the box, which covers a lot of product. These two protocol standards are similar but different enough to be...different, and supporting both means you can mix and match sensors as you discover what works for your projects.
* Large Ecosystem. There is a wide variety of products and sensors that work with SmartThings, for better or worse. Anything that needs monitoring in a house, there's probably a compatible device that will do it. And in the larger stew of proprietary platforms, a lot of interoperability exists out of the box.
* Low cost of entry. The [hub costs $99][hub]. A [starter kit runs $250][kit]. That's not too shabby. You'd definitely spend at least that to [roll your own][] solution.
* Open development platform. SmartThings gives you a Java/Groovy-based development platform to write your own automation jobs and entire devices. With a sandboxed stripped-down set of classes and a pretty well-documented API, you can set out to make even the most complicated tasks automatic. This is a lot of what attracted me to the platform in the first place; I only wish some "Smart Apps" could be aware of the others.

# Its Role 

I have a SmartThings hub and a handful of sensors:

* A SmartThings-branded Temperature/Humidity sensor (ZigBee) in each of the two upstairs bathrooms where we're most likely to bathe.
* An [Aeon Labs][aeo] [Multisensor][sensor] (Z-Wave) that also detects motion and light levels. This one floats depending on what I'm playing with. Right now it's in a downstairs bedroom to see how the comfort levels differ between upstairs and down.
* I also have a few switches for turning things on and off. Since Z-Wave devices create a mesh network from themselves, it's a good idea to have a few hard-wired switches around since they also act as repeaters. Battery-powered sensors do not.

I intend to add more sensors. Samsung makes an [open/close sensor][msensor] that also does temperature (but not humidity, unfortunately) that would be handy for most rooms; a water leak sensor in the utility room near the water heater would be a good idea; additional motion or multisensors could also be useful.

What I don't want to do is try to build a redundant "alarm" system out of SmartThings sensors. I have that handled, in a UL-listed professional kind of way. A cloud-based home automation approach to security would be naive, but there might be some corner cases I can cover with the right tools.

Regardless, I expect the SmartThings hub and devices to play a key role in accomplishing most if not all of the [Use Cases][] I aim to address.

# Cross-Platform 

Some of the other tools I have that integrate with SmartThings include:

* Netatmo Weather Station
* Belkin Wemo
* Philips Hue
* Amazon Echo

As I mentioned above, SmartThings gives you a platform on which to write your own integration. Really, if you have a thing with an API you can access via URLs you can write a SmartThings Device for it. I might actually do this with my whole-house audio system but first I need to bolt an API onto its serial port.

[Use Cases]: /use_cases/
[4 years]: https://www.engadget.com/2016/06/21/samsung-invests-in-internet-of-things/
[SmartThings]: https://www.smartthings.com
[Z-Wave]: https://en.wikipedia.org/wiki/Z-Wave
[ZigBee]: https://en.wikipedia.org/wiki/ZigBee
[aeo]: http://aeotec.com
[hub]: http://amzn.to/28WalZ5
[kit]: http://amzn.to/28QMyuV
[sensor]: http://amzn.to/28QMICA
[msensor]: http://amzn.to/28NtoTE
[roll your own]: http://www.openhab.org

# More To Come

This is a very broad overview of SmartThings, and there are a lot of facets I might want to discuss.

# Pages About SmartThings

<ul>
{% assign sorted = site.smartthings | sort: 'moddate' | reverse %}
{% for item in sorted %}
  <li>
    <a href="{{ item.url }}">{{ item.title }}</a>
    <span class="date">{{ item.moddate | date: "%B %-d, %Y"  }}</span>
  </li>
{% endfor %}
</ul>

