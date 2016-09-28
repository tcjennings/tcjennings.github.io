---
title: A Tale of Two Internets
layout: post
tags: the_house the_internet
---

I've had two Internet setups at [the house] so far, with a third and -- I hope -- a fourth coming soon.

# Long-Term Evolution, Short-Term Solution

As the neighborhood was new construction, options for Internet were limited at first, as only AT&T had installed any infrastructure. Because of city Planning & Development confusion, those options were between nothing and nothing. I needed something to bridge the gap while we worked out the confusion, so I turned to a Sprint LTE hotspot.

Running a hotspot connected via USB to my computer, I ran [PFSense][] in a virtual machine and used it as a firewall between my network and the public Internet. This worked out a lot better than I thought it would, but with significant caveats.

1. I had to reboot the hotspot every one or two days when it would inexplicably lose connection. Not being intended for 24/7 operation, I suppose I can't fault it too much, but any devices with telemetry would alarm when Net access went away, so that was irritating.

2. Data caps on LTE networks are significant. I ended up getting a "family plan" of 20GB "shared" bandwidth and even without streaming media we ate that up every month. We're a bandwidth-hungry household, and it was not pleasant to go on a diet.

# DS(Hel)L

Eventually and finally, my address became correctly coded by AT&T engineering to be eligible for service, but the catch was that the service was 768k DSL. That's roughly the speed of a T1 line which in less saturated times was sufficient to provide Internet access to an entire office of workers while hosting the company's web site. These days, not so much.

Still, I jumped at the chance for the hookup, since I knew it was my "wedge" service. Being only the second household on the block, I could still point to the *first* household and their 12 mbps service. If they can have it, why can't I?

So I signed up. And they came out and did the thing, and it took a long time to get the hookup made and working, but eventually the little modem lit up and I could get an IP address and make the sobering mistake of hitting a speedtest server. I think I ended up with something just over 800 kbps. See that? Underpromise and overdeliver, that's the way you do it.

Regardless, I had my wedge. It was time to start making phone calls, being escalated, pleading my case, etc. In time I was hooked up with the "Office of the President," a phrase which seems to carry some weight in AT&T but not as much as the name suggests. They're certainly the people who can Get Things Done, though.

Eventually, my address ended up being coded as eligible for a 48 mbps U-Verse service, which again I jumped on. This required more truck rolls and outside cable plant work and eventually we determined that I could achieve the 12 mbps speed tier, just like the neighbor.

In the meantime, I had an order number from Time Warner that they had promised was delivered to their Engineering department for build-out (remember, they had no copper in the ground yet), and crews from Google Fiber were installing their conduit throughout the hood.

So I took what I could get from AT&T and figured things would progress and I could ditch them, even if it meant paying an early cancellation fee (they could have it!). (Not worth mentioning in detail is that other significant distractions came along that made the speed of my Internet service rather unimportant.)

# One Year Later

Google Fiber installed their conduit, and I'm pretty sure they installed fiber in that conduit, but they still haven't done the rest of their infrastructure or made any progress in delivering service.

That order with Time Warner didn't go anywhere, and it's only been in the last couple of months that they came through and installed their copper. I knew it must be ready for installation because their sales rep left a business card and a channel line-up card on my door handle, as though I'd never tried to set up service before.

My AT&T U-Verse service is plugging away at 16 mbps, according to [fast.com](http://fast.com), which is "enough" for our streaming needs but the anemic 1.5 mbps upload speed is too easy to saturate, which brings the whole network to a halt.

The whole time I've been using AT&T's modem without my own router, firewall, or UTM (Unified Threat Management: think anti-spam, anti-virus, anti-etc.) layer between it and my network because, well, it was meant to be temporary. I've only got one Wi-Fi access point, one of the old flat Apple AirPort Extremes, which despite being as central as possible still can't quite paint the corners of the house with useful signal.

I have a new order in with Time Warner Cable to deliver 200 / 20 mbps service, which is much, much faster than AT&T's offering and will be much cheaper as well (for the first year, anyway, so...come on, Google!).

In anticipation, I got my own cable modem and a router to put between it and my network. At $90, the modem will pay for itself after 9 months of avoiding the cable company's $10/month modem rental fee. At $50, I expect the router to be more than capable of handling the 200 mbps throughput, though it won't have the UTM features of something like PFSense.

# The Modem

Time Warner publishes a [list of modems](https://www.timewarnercable.com/content/dam/residential/pdfs/support/internet/twc-compatible-modems.pdf) it claims are compatible with its service, sorted by manufacturer, Wi-Fi support, and top speed. The modems generally fall into three speed categories: 50 mbps, 100 mbps, and 300 mbps. Because I'm going for a 200 mbps service (the 300 mbps level is, frankly, overkill for the downlink and offers the same 20 mbps uplink as the 200), I'd still need to target a 300 mbps modem.

Unfortunately, I found that the majority of the highest-speed modems aren't any cheaper than Time Warner's rental rate for a year. I settled on the [Motorola MB7420](http://amzn.to/2dlXS0r) which at $90 at least doesn't break the bank.

There's very little to the thing: power, coax, and Ethernet. Plug in the three of them and you should have Internet. We'll see. I did power the thing up to make sure it's not DOA, but before I do anything useful with it, the guy needs to do the thing with the hookup.

If the Time Warner install is anything like the AT&T install, that means he'll run some wire across the ground from the utility pedestal in the yard to my house, then some time later a guy with a sod cutter will show up and bury it.

# The Router

Since the modem doesn't have a built-in router, firewall, or NAT capability, I needed something to put between it and my network.

I could use my AirPort Extreme, but I plan to retire it in favor of some newer, beefier Wi-Fi access points.

I could use an old Linksys WRT54G with DD-WRT on it, but it wouldn't route anything close to the speeds I'll be paying for, so that's out.

I could use PFSense, which is a decent piece of software whose capabilites are equal to the hardware you buy to run it. For price/performance -- not to mention performance/watt -- it doesn't really make an attractive case. I would have to run a virtual machine like I did before, which would tie Internet access to my computer (not something I want to do), build a new dedicated platform for it (too expensive, and too power-hungry), or buy a PFSense appliance (too expensive, but much more power-conservative). So PFSense is out.

The market is flooded with SOHO routers, and the current design philosophy seems to require they all look like [alien spacecraft](http://amzn.to/2daEfvv). They also all seem to be all-in-one router/WiFi boxes and I don't need or want that. I'd rather have discrete components so I can upgrade/replace/extend by function and feature.

So I decided to start small and get the smallest baby router in [Ubiquiti][]'s lineup, the [EdgeRouter X](http://amzn.to/2daUyah), which has among other benefits a complete CLI-driven network operating system. As a guy who's been Cisco and Juniper certified, built and maintained enterprise and service provider networks, this stupid little detail is a real selling point for me.

It also has 5 gigabit ports and supports hardware acceleration of some of its core routing functions (a feature you'll never get in a PFSense box). I'll do some brief performance testing of my own once everything's installed and ready to go, but I expect it to be plenty sufficient to drive a 200 mbps cable modem, and I'll probably have to replace it when Google finally delivers some gigabit fiber service.

# The Wi-Fi

Just a quick note about my Wi-Fi plans, which will be a post of its own: I'm getting an Ubiquiti [Unifi Pro](http://amzn.to/2cCuq5y) access point to replace my AirPort, and I'll get a second one if necessary for coverage. There's no real benefit to matching the access point and router brands, it's just that these guys seem to have good Enterprise-level products at home-network prices. The ER-X router has passive Power Over Ethernet capability on one of its ports, but it's not the kind of POE the Unifi Pro can make use of, so that potential synergy is moot.

# Conclusion

So that's the stage set. We'll see how this all works out later this week when things start getting installed.

[Ubiquiti]: https://www.ubnt.com/
[the house]: /the_house/
[PFSense]: https://www.pfsense.org
