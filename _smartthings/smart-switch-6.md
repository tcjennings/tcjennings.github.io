---
title: Aeon Labs Smart Switch 6
layout: page
tags: [smartthings]
moddate: 2016-06-27
---
<img style="float: right;" src="/images/smart-switch-6/unbox.jpg" width="320 px" />

When it comes to switching things on and off via [Z-Wave](https://en.wikipedia.org/wiki/Z-Wave), you don't have a lot of options. You can buy an appliance that has Z-Wave functionality built in, you can install a Z-Wave module behind your in-wall switches or replace the switch entirely, or you can use a plug-in module.

The latter option is the least permanent and requires no electrical work. You don't have to shut off the breaker for the circuit you're working on and inevitably discover that no matter how comprehensive your panel labelling is it doesn't seem to cover *this* particular location so you resort to the flip-and-yell method of circuit identification.

I picked up a pair of these [Smart Switch 6][] plug-in controllers by [Aeon Labs][] for $50 each. In addition to simple on/off control, these switches track electrical usage, provide an always-on USB charging port, can serve as a night-light, and perhaps most importantly extend the Z-Wave mesh network by acting as a repeater.

<img style="float: right;" src="/images/smart-switch-6/vs.jpg" width="320 px" />

# Unboxing

Look. You open the box and you take the thing out.

# Getting Into It

The brochure site lauds these devices for their small size. They are small, and also heavy; they feel built like a tank. The third iteration of Aeotec's smart switch design, they are much better designed than their predecessors. Here I have the Smart Switch 6 next to two Belkin WeMo plug-in switches: the [Insight Switch](http://amzn.to/28YyE76), which has the same usage-tracking feature, and the ordinary [plug-in switch](http://amzn.to/28YKksw), which is way too large for its own good.

The Smart Switch 6 packs a lot more functionality into a smaller package and has a nicer build quality than the Belkin modules. The Belkin Insight Switch does have a convenient touch-button on its top, which the Smart Switch 6 lacks; instead, the Smart Switch 6 has a little toggle in the front lower-right corner which is also used to include the device in the Z-Wave network.

<img style="float: right;" src="/images/smart-switch-6/bottom.jpg" width="160 px" />

<img style="float: right;" src="/images/smart-switch-6/top.jpg" width="160 px" />

Despite the small size, the Smart Switch 6 doesn't play nice with a standard outlet. Installed into the bottom outlet, the module blocks the top outlet. Luckily the inverse is not true, so these modules are only suitable for installation in the top outlet. In roomier applications, I'm sure they nestle together quite nicely.

# Integrating The Thing

Pairing the switches with my [SmartThings][] hub was deceptively straightforward and easy. You tell the SmartThings app to add a device, press the button on the front, and wait. If all goes well, you'll see it get discovered. If not, maybe you need to move closer to your hub.

The modules report to SmartThings as a "Z-Wave Metering Dimmer," which is not quite right as these modules do not support dimming. I installed a community device definition for this module from [Robert Vandervoort][github] that better represents the device's capabilities. As this and other device definitions are open-source, I can fork and modify the device if I need to.

Once set up, the module reports state information for "switch" (of course), "energy" and "power". These last two are similar, but "power" reports the current current draw (ha!) in Watts, and "energy" reports the cumulative energy consumption in kWh. Robert's device code also supports setting three preferences for the device, including setting the LED ring on the device to report current draw or act as a night-light. When a nightlight, the color of the ring can be controlled using a color-picker tile in the SmartThings app.

# Exclusion

If you can add a device to your network, you need to be able to remove it, too. In Z-Wave networks, this is called "exclusion." SmartThings has a button in the app to set the hub into "Exclusion Mode," in which it listens for devices requesting removal from the network.

The instructions for the Smart Switch 6 say to "press the action button" to request exclusion, but this is unclear. A momentary press of the button turns the module on or off. To get exclusion to work, I had to press *and hold* the action button until the LED ring started a gradient color loop, and after a few refreshes of the SmartThings app the device disappeared from the list of Things.

# Verdict

The Aeon Labs Smart Switch 6 is a nifty piece of tech in a small package. It feels like it's built like a tank and it adds a lot of functionality to a Z-Wave network. The always-on USB port is a nice touch, and it can replace a USB-charging wall-wart and add Z-Wave switching at the same time. However, it puts out a relatively anemic 1000 mA of juice, so it's on par with a phone charger. It would take a long time to charge a tablet and doesn't have enough punch to supply something like a Raspberry Pi.

At $50 it's priced similarly to Belkin's WeMo Insight Switch, which provides similar energy-tracking features, but is too expensive to justify including many of them in your network. Z-Wave nearly begs for a device in every electrical box just for range and mesh networking, but the Smart Switch 6 should be a luxury, not the workhorse of your network.

Including the Smart Switch 6 in the network could be tricky for beginners. While it is discovered and included quickly and easily, the default SmartThings device profile doesn't expose the full range of features and includes a knob for a state that is unsupported by the device (dimming).

For advanced users, finding and implementing a custom device profile is easy enough, and it can serve as a case study for dissecting how a custom device can be built from manufacturer's engineering document; the strength of a platform that promotes open-source like SmartThings comes through.

My initial experience with these modules was not positive. Within hours of setting them up they became INACTIVE and I had to move one of them closer to the Hub, so there may be a range issue. The only real solution to that is to add more Z-Wave devices to the network, and I don't have a great deal of them installed yet. Regardless, a plug-in module isn't that handy if you can't plug it in where you need it.

[Smart Switch 6]: http://amzn.to/294RsWZ
[Aeon Labs]: http://aeotec.com/z-wave-plug-in-switch
[SmartThings]: /the_tools/smartthings.html
[github]: https://github.com/robertvandervoort/SmartThings/tree/master/Aeon%20SmartSwitch%206