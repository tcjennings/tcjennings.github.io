---
layout: post
title: The Dashboard (Part I.V)
tag: elk-stack
---

# Kibana and Memory

[Last post] I left off having just set up Kibana. In the meantime I've added some more data to my Elasticsearch database and created some visualizations and dashboard in Kibana. Then my Raspberry Pi ground to a halt when Node decided to use all the memory and began swapping like mad (not a great thing to happen on any platform, but doubly so when your storage is an SD card).

Bug reports like [this][1] and [this][2] illuminate what is probably the problem. The Raspberry Pi is a fairly low-resource environment, and I already gave a full 50% of the available RAM to Elasticsearch.

[Last post]: {% post_url 2016-06-21-The-Dashboard %}
[1]: https://github.com/elastic/kibana/issues/5170
[2]: https://github.com/elastic/kibana/pull/5451 "This is actually a merge request, not a bug report"

I've edited my `/etc/default/kibana` file to include the line:

```
NODE_OPTIONS="--max-old-space-size=100"
```

So that when Node/Kibana is running the Node garbage collector will be more aggressive about cleaning up the heap.

# Disabling Raspberry Pi Swap

The other thing I want to do is disable swap entirely. Raspian out of the box gives you a 100MB swap file (not swap partition), which isn't much in the first place. I shouldn't miss it.

This swap file is configured from `/etc/dphys-swapfile`:

```
pi@raspberrypi ~ $ cat /etc/dphys-swapfile 
CONF_SWAPSIZE=100
```

Not much to it, but there's your 100MB of swap, which is confirmed with `free -h`:

```
pi@raspberrypi ~ $ free -h
             total       used       free     shared    buffers     cached
Mem:          925M       613M       312M        15M        41M       206M
-/+ buffers/cache:       365M       560M
Swap:          99M         0B        99M
```

Getting rid of this swap file is a three-part process:

```
# Turn off the swap file
sudo dphys-swapfile swapoff

# Delete the swap file
sudo dphys-swapfile uninstall

# Tell the swap file not to come back
sudo systemctl disable dphys-swapfile
```

Check the work with `free -h` again and see there's no longer any swap available:

```
pi@raspberrypi ~ $ free -h
             total       used       free     shared    buffers     cached
Mem:          925M       621M       304M        15M        41M       209M
-/+ buffers/cache:       370M       555M
Swap:           0B         0B         0B
```

None of this is permanent. I can turn the swap back on by reversing the `dphys-swapfile` commands above if I ever end up needing swap.
