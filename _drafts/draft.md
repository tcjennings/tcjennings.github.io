# The Use Cases

You talk about "Smart Homes" and I think most people jump to the gimmicky stuff. Bill Gates famously built a smart mansion in the 1990s where visitors could wear a special electronic pin that would tell the house how to arrange the rooms you happened to enter. Dim the lights the way you like, play the music you want to hear, put the art you prefer up on the screens. That's gimmicky and worthless. And I want it.

But I have more practical use cases, most of which center around managing comfort within my home and automating the kind of tasks that make home ownership tedious. It'd be nice to know when something's wrong and get a proactive heads-up about it. I want to treat my house like a platform. Things that need doing need to get done, so they have to be tracked. The mortgage company didn't provide a helpdesk so I'll have to figure it out myself.

# The House

My house was built in 2015 to LEED standards. I think it earned LEED Silver certification. A lot of what that means was baked into the construction process and doesn't have a daily impact on my life, but there is a particular appliance involved that is central to managing the air quality within the home. The lighting is exclusively LED (except for a lamp or three with legacy CFLs in them), and the appliances are all Energy Star-certified, as far as I know. The HVAC is a 95% efficient heat pump+gas furnace. The windows are all engineered with UV-filtering materials to keep heat out and save my stuff from fading; the exception is a large south-facing architectural detail window that earned some LEED points with its passive-solar effect.

I put not-quite-enough copper into the walls to support most networking tasks, but that's largely independent of home automation gear, which tends to be wireless anyway. There's lots of places to plug in a PC or a game console, hook a TV up to antenna or cable, or plug in a pair of speakers.

There's a security system with your typical array of magnetic reed sensors for external doors, motion detectors, fire and smoke detectors, and keypads. They're all wired in and separate to the kind of home automation I'm going to implement, but it would be nice to have a notification or status bridge between the security panel and its devices and the home automation platform. After all, if I want to do something when a door opens, I don't want redundant sensors on that door to figure it out.

## The ERV

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

Luckily the ERV has a low-voltage wiring block I can use to my advantage. By creating a connection between two terminals I can turn it on; by creating a connection between two other terminals, I can switch it to high speed.

Specifically, I can use dry contact relays to make these connections:

* Jumper between ON and LOW initiates low speed operation.
* Jumper between ON and HI initiates high speed operation.

I can make do with that. Some of the fancier wall controls have as many as 5 speed settings, but for my purposes switching between Low and High seems fine.

In a purely mechanical world, I could use a 3-position switch to make these connections, where the ON terminal is the common line and I switch it between LOW and HI (with the third position being OFF). But I do not want a mechanical solution, I want a smart solution.

## The Sprinklers

My house is not on a large lot. My previous house was, and while a big backyard with mature trees sounds like a great thing on paper, it's not as great in real life. I don't have dogs or children, so I didn't really need the space. It was just a thing I had to take care of.

The new house has a sprinkler system, but unlike most houses with sprinkler systems, mine is connected (via a pump) to a rain barrel. No, scratch that. A giant 530-gallon [rain tank][] that stands 6 feet tall. The tank is connected to one of the gutter downspouts, and the way the gutters are arranged, the tank is fed by about half of the available roof area. The tank of course has an overflow for when it's full.

This might not have been a great idea, but the tank is not connected to plumbing, and the sprinklers are exclusively supplied by the tank. This eliminates the need for backflow valves and the requisite inspections thereof, but introduces the wrinkle that if it doesn't rain, I have to fill the tank with a garden hose in order to water the lawn.

Maybe if I went for LEED Platinum I would have an entirely xeriscaped landscape without any turf to water twice a week.

The process for filling the tank is as dumb as can be: I have to figure out if it's empty, and if it's likely to rain soon. If it is, and it's not, then I have to run the hose for about an hour to fill the thing up. That's really a guess; I haven't actually timed it, nor have I really done the work to figure out how many "standard" waterings I get out of a full tank.

If I want to make this process smart, I'm going to have to figure out the level of water in the tank and the weather forecast for the next several days.

The other side of this use case is whether to run the sprinklers at all. If it rained recently and won't rain again for a while, I can probably skip a day and use my water reserve for a later watering. On the other hand if it rained recently and will rain again in a few days, I can afford to dump what's in the tank into the lawn since it'll be refilled by nature in time for the next watering need.

Of course my sprinkler controller is a builder-grade something or other with a user interface that was probably written in COBOL by some retired IBM coder. I can't figure out how it works just by looking at it, and every time I think to look up the documentation online I forget what it's called.

[ERV]: https://en.wikipedia.org/wiki/Energy_recovery_ventilation
[rain tank]: http://www.bushmanusa.com/530-gallon-slimline-rain-tank