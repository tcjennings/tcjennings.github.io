---
title: The Sprinklers
layout: page
logline: The grass needs watered, but let's not be wasteful about it.
---
My house is not on a large lot. My previous house was, and while a big backyard with mature trees sounds like a great thing on paper, it's not so great in real life. I don't have dogs or children, so I didn't really need the space. It was just a thing I had to take care of.

The new house has a sprinkler system, but unlike most houses with sprinkler systems, mine is connected (via a pump) to a rain barrel. No, scratch that. A giant 530-gallon [rain tank][] that stands over 6 feet tall. The tank is connected to one of the gutter downspouts, and the way the gutters are arranged, the tank is fed by about half of the available roof area. The tank of course has an overflow for when it's full.

This might not have been a great idea, but the tank is not connected to plumbing, and the sprinklers are exclusively supplied by the tank. This eliminates the need for backflow valves and the requisite inspections thereof, but introduces the wrinkle that if it doesn't rain, I have to fill the tank with a garden hose in order to water the lawn.

Maybe if I went for LEED Platinum I would have an entirely xeriscaped landscape without any turf to water twice a week. Well, that's the dream anyway. I can't sit here and talk about how "green" and eco-conscious I am when I run the same two-stroke small-engine lawnmower every week as the next guy.

The process for filling the tank is as dumb as can be: I have to figure out if it's empty, and if it's likely to rain soon. If it is, and it's not, then I have to run the hose for about an hour to fill the thing up. That's really a guess; I haven't actually timed it, nor have I really done the work to figure out how many "standard" waterings I get out of a full tank.

If I want to make this process smart, I'm going to have to figure out the level of water in the tank and the weather forecast for the next several days.

The other side of this use case is whether to run the sprinklers at all. If it rained recently and won't rain again for a while, I can probably skip a day and use my water reserve for a later watering. On the other hand if it rained recently and will rain again in a few days, I can afford to dump what's in the tank into the lawn since it'll be refilled by nature in time for the next watering need.

Of course my sprinkler controller is a builder-grade something or other with a user interface that was probably written in COBOL by some retired IBM coder. I can't figure out how it works just by looking at it, and every time I think to look up the documentation online I forget what it's called. (I exaggerate; it's a [Hunter X-Core][]. But it really is inscrutable.)

It's clear that this won't suffice for really making my dumb sprinklers smart. Of course, I have no idea how standardized sprinkler controllers are. I know there are "smart" controllers on the market, but I haven't yet done the research to figure out my next steps here.

# Posts About Irrigation

{% for post in site.tags.irrigation %}
<ul>
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date">{{ post.date | date: "%B %-d, %Y"  }}</span>
  </li>
</ul>
{% endfor %}


[rain tank]: http://www.bushmanusa.com/530-gallon-slimline-rain-tank
[Hunter X-Core]: https://www.hunterindustries.com/irrigation-product/controllers/x-core
