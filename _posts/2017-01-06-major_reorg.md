---
title: Major Re-org
layout: post
tags: about
moddate: 2017-01-06
---

For the new year, I've decided to put some time into reorganizing this site. When I started it in 2016 I was excited about creating a braindump around my smart home hobby and quickly decided on Jekyll and GitHub Pages to do it. I focused on content (such as it is) instead of functionality and ended up doing things in a way that I was not happy with the long term, which ended up discouraging me from creating the content in the first place.

One of the mistakes I made was not appreciating Jekyll's types of content and the organization thereof: posts, pages, and collections.

The easiest (i.e., the lowest barrier to entry) was to make everything a "post" which gave the site a blog-like affect. That's fine on the face of it, but it rubbed me wrong for a couple of reasons. One, to me a "blog" entry is a temporal thing, which is why entries are dated and sorted according to that date. My pieces of content -- besides meta posts like this -- aren't temporally restricted and should just *be*. That might not mean much on the surface, but under the hood the fact that I was making everything a post meant that the date became part of the content's identity -- it was in the filename, and if I wanted to make a crosspost to it the date was part of that, too. If I updated a page, and changed the date so it popped to the top of the site, that meant all the crosspost links would be broken since the filename would change. And etc. This is all my fault for not doing Jekyll correctly at the outset.

One benefit was that it was easy to create the topic navigation strip at the top of the site, using post "tags" to organize the content according to one of the canned topics. Again, "easy" but not optimal, because I didn't put the 20 minutes into grokking Jekyll's "collection" concept.

So I'm reorganizing the site. Even though it might not look dramatically different, there's a lot of changes behind the scenes that will make it easier to update and maintain.

This is not a re-*design*, obviously. I am looking at some Jekyll themes that are just as simple but have a few extra slots for content, but I'll tackle that a little later. For now, the default theme is readable and renders well on mobile.

Conceptually, I am organizing the site like this:

1. Blog feed. Date-stamped posts that are *about* the site (like this one) will be "posts" that appear in the blog feed, which is this main home page. Posts will talk about site changes, call out new content, maybe be a place where I post temporally-relevant things about smart home news. Idea and technical articles/pages will not appear in the blog feed.

2. Collections. The top navigation row of topics is the entry point to different collections that I think is logically organized from general to specific:

    * First Principles, which is a new category of pages I'm going to launch, in which I'll try to describe and enumerate the reasons behind (my ideas of) smart home automation, which will accrete to a big-picture roadmap.
    * The House, a collection of pages that describe the house itself, and systems inherent to it.
    * The Tools, a collection of pages (really a collection of collections) that describe the different tools or technologies I encounter or use in the course of pursuing this site's mission.
    * Use Cases, a collection of collections that describe the tasks for which the tools are put to use. These may be general or specific, simple or complex, useful or trivial. I think it's the heart or meat of the site, only possible through the building blocks of first principles and tool use.
    
3. About. The about page is the about page. Nothing much changing there.

I figure navigation for a visitor would go something like this:

1. See new stuff posted about in the blog feed.
2. Explore the collections from left-to-right as interests dictate.
3. Crossposts drive visitor to other pages or offsite (hopefully to return).

We'll see. In the meantime, there might be some incomplete sections or broken links as I work through the re-organization.