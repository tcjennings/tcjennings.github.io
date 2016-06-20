---
layout: page
title: Use Cases
---
You talk about "Smart Homes" and I think most people jump to the gimmicky stuff. Bill Gates famously built a smart mansion in the 1990s where visitors could wear a special electronic pin that would tell the house how to arrange the rooms you happened to enter. Dim the lights the way you like, play the music you want to hear, put the art you prefer up on the screens. That's gimmicky and worthless. And I want it.

But I have more practical use cases, most of which center around managing comfort within my home and automating the kind of tasks that make home ownership tedious. It'd be nice to know when something's wrong and get a proactive heads-up about it. I want to treat my house like a platform. Things that need doing need to get done, so they have to be tracked. The mortgage company didn't provide a helpdesk so I'll have to figure it out myself.

That doesn't mean I won't put together gimmicky show-off automation toys, too. 

{% for thepage in site.use_cases %}
{% if page.title != thepage.title %}
<a href="{{ thepage.url | prepend: site.baseurl }}">
        {{ thepage.title }}
</a>

<p class="post-excerpt">{{ thepage.excerpt | truncate: 160 }}</p>

{% endif %}
{% endfor %}      
