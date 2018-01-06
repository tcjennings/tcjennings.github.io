---
title: LED Pixels I
layout: page
tags: the_tools lighting
---

In this post I describe operating a string of individually-addressable [LED pixels][] using [Open Lighting Architecture][] on a [Raspberry Pi][] through [Docker][]. It's jargon-heavy but conversational in tone.

# Technology and Hardware

LED light strips are nothing new; they're ubiquitous and cheap. You can get "smart" ones from [Hue][]; monochrome or RGB; indoor or outdoor. A strip of LED pixels, on the other hand, is a different animal because every LED lamp in the strip is individually-addressable, which allows you to do things with it you can't do with an ordinary LED strip. A common use case might be a 16x16 or larger grid of LED pixels which are animated in some fashion[^1].

Serial Peripheral Interface ([SPI][]) is a lightweight serial networking bus that is often used by the chips controlling readily-available LED pixel strips. Each LED is a separate device on the bus, which allows one to target data for individual lights. SPI is the hardware language they speak.

The Raspberry Pi is a low-cost and low-power general purpose computer that can run Linux. It also has an array of GPIO (General Purpose Input/Output) pins that include an implementation of SPI.

Docker is a way of running software on Linux and other operating systems. It can be described as "chroot with networking" but for the purposes here it allows one to run a piece of software in an isolated environment with everything it needs to operate.

Open Lighting Architecture (OLA) is an open-source lighting control framework. Traditionally, the [DMX][] protocol is used for lighting control, and OLA implements a DMX framework with plugins for other protocols such as SPI and ArtNet (a network implementation of DMX), which are the two protocols I'm using in this activity.

Other technologies used here will include [Node.js][], a JavaScript runtime engine and some JavaScript code.

## Hardware

The LED pixels I'm using are a string of 25 WS2801-based pixels bought from [Adafruit](https://www.adafruit.com/). The "WS2801" refers to the IC that drives each pixel in the string; it implements SPI and drives the LED.

The Raspberry Pi I'm using is a Pi 3 running the latest Raspbian operating system and Docker. I have a couple of services already active such as CollectD, InfluxDB, and [Grafana][].

Both the pixel string and the Pi are powered by a 5V DC power supply. I happen to power the Pi from a nearby high-amperage USB port, and for the LEDs I am using a 2.5 amp 5V power supply I had laying around.

## Operating System Tuning

Before we get started, know that the Raspberry Pi's support for SPI is not enabled by default. You have to enable the SPI kernel module first, which is easily done using the [`raspi-config`](https://www.raspberrypi.org/documentation/configuration/raspi-config.md) utility. In its "Advanced Options" menu there is an option to enable the SPI interfaces. After doing so and rebooting, the SPI interfaces are present on the system, in the `/dev/` filesystem:

```
ls -l /dev/spidev*
crw-rw-rw- 1 root spi 153, 0 Sep  1 21:28 /dev/spidev0.0
crw-rw-rw- 1 root spi 153, 1 Sep  1 21:28 /dev/spidev0.1
```

# Open Lighting Architecture

The goal is to use the OLA server software to control the LED pixels through the Pi's SPI pins, and to use ArtNet to talk to the OLA server over my home network.

Since OLA is a lighting control framework, it works on the conceits of the DMX protocol -- with it you can configure "Universes" that consist of up to 512 channels or addresses, which is how things work in DMX. In OLA, each configured Universe has an input and an output, so different Universes on the same server can talk to different kinds of devices; SPI in one universe, for instance, and actual DMX using a USB adapter in another universe.

OLA is open-source software, which means it's free (as in beer) and Free (as in speech). It also means it's not the most user-friendly or well-documented software in the world. Written in C++, OLA can be built from its source code or installed from binary packages for some Linux flavors (though these packages appear to lag behind the current release version). They even distribute a version of Raspian custom-built to run OLA.

I am running OLA in a Docker container that I wrote. Anyone with a Raspberry Pi and Docker can download and run my OLA container image without any additional knowledge or hassle. Docker containers encapsulate the entire environment needed for the software to run, so it will run the same way everywhere. It's like a fat binary distribution.

## Docker

A brief aside about Docker. My [Docker image for OLA](https://hub.docker.com/r/tcjennings/rpi-openlighting/) is public and anyone can download and use it, and the instructions for its creation are in GitHub so can be reproduced by anyone with an interest to do so. I have used the most recent release of the OLA software (0.10.5).

Docker by itself can be complicated to understand, but one key way to ease its use is through orchestrators that define sets of containers to run. I use [Docker Compose][] as an orchestrator to manage Docker services on my Pi, so I already have a "smarthome" stack of services defined. I can easily add another for OLA. By itself, a Compose file for OLA might look like this:

```
version: '2'
services:
  ola:
    image: "tcjennings/rpi-openlighting:latest"
    ports:
      - "9090:9090"
      - "9010:9010"
      - "6454:6454/udp"
    volumes:
      - ./ola/config:/etc/olad
    devices:
      - "/dev/spidev0.0:/dev/spidev0.0"
      - "/dev/spidev0.1:/dev/spidev0.1"
    restart: always
```

If a file named `docker-compose.yml` with the above contents is placed in a directory, then the command `docker-compose up -d` will start every service defined in that file (the "-d" means "daemonize" or run in the background).

The configuration shown above defines a service named "ola"; specifies that the image for it can be fetched from Docker Hub at the given address, that the container will use ports 9090, 9010, and 6454 (UDP); that it should have access to the SPI devices; and that the directory `/etc/olad` inside the container should be mapped to a particular location on the host (to store the OLA configuration).

Once I've started OLA, then I can reach its UI on port 9090 with a web browser. Port 9010 is an RPC port and 6454 is the port used by ArtNet.

## Initial OLA Configuration

My intent is to set up a Universe with an Art-Net input and an SPI output so I can send DMX data over the network to OLA and drive the LED pixels. Before trying anything fancy, I want to make sure that OLA works properly within my Docker container.

First, I'll set up a Universe with an Art-Net input and a Dummy output (using the OLA plugin named, appropriately, "dummy" which by default will just print what it receives to standard out).

With no additional configuration, I should be able to direct Art-Net commands to my Raspberry Pi and see the Dummy plugin produce output. On the Pi I want to make sure I'm looking at the OLA server's output; since it is running in a Docker container, I can see this with the command `docker logs -f <name>` where `<name>` is the name of the container.

From another computer on the network, I want to send some Art-Net commands. One easy way to do this is to run a Docker container with the Node.js software in it. On my Mac I issue the following commands to start a Node container, install an Art-Net package and run some very simple commands:

```
docker run -it --rm node:6 bash
npm install artnet
node
var artnet = require('artnet')({host: '192.168.1.XXX'});
artnet.set(1,255);
.exit
exit
```

In the above, the commands send an Art-Net command to the specified host (my Raspberry Pi) that says to set channel 1 to a value of 255, then close the connection. Subsequently I issue an exit command to both node and the bash shell, which causes the container to stop and be removed, leaving no trace of the software on my system.

On the Pi, where I have a shell with the `docker logs` command active, I see the output from the dummy plugin in the OLA server's output:

```
olad/PluginManager.cpp:195: Trying to start Dummy
olad/plugin_api/DeviceManager.cpp:105: Installed device: Dummy Device:1-1
olad/PluginManager.cpp:200: Started Dummy
olad/PluginManager.cpp:195: Trying to start ArtNet
olad/plugin_api/DeviceManager.cpp:105: Installed device: ArtNet [172.18.0.2]:2-1
olad/PluginManager.cpp:200: Started ArtNet
olad/plugin_api/PortManager.cpp:119: Patched 1-1-O-0 to universe 0
olad/plugin_api/Universe.cpp:522: Full RDM discovery triggered for universe 0
olad/plugin_api/PortManager.cpp:119: Patched 2-1-I-0 to universe 0
plugins/dummy/DummyPort.cpp:123: Dummy port: got 2 bytes: 0xff 0x0 
```

The last line shows the Dummy plugin receiving the Art-Net message, where "0xff" is the hex code for the value 255. If I repeat the test with other values, I should expect them to show up at the server in the same way.

Another way to verify operation is through the OLA UI, in the Overview tab for the Universe, where each of the available 512 channels is listed along with the associated value.

If I issue an Art-Net command like:

```
artnet.set(1,Array(512).fill(127));
```

The result will be that each of the 512 channels in the Universe will be set to the value 127. You start to see the utility and flexibility available from even a very simple Art-Net program.

## An OLA Configuration Note

The way OLA works is that it does not save its configuration automatically as changes are made, nor is there a discrete "save" button. Instead, it saves its configuration on "shutdown" which is meant to be done gracefully using the "shutdown" button in the Web UI, for instance.

If the process stops ungracefully (because you yank the power cable, or just issue a `docker stop` for the container), the universe probably won't be saved. Once saved, a number of files in the configuration directory are updated.

# Hardware Hookup

The LED pixels have four wires between each module: Green, Yellow, Red, and Blue. Red and Blue are the +5VDC and Ground, respectively. The Green wire is the Serial Clock line and Yellow is the Serial Data line. Pretty simple: 2 power lines, and 2 SPI lines.

The Raspberry Pi has a bunch of pins, but I am only interested in 3 of them. The run of SPI pins are physical pins 19 (MOSI), 21 (MISO), and 23 (SCLK). Of these I only need MOSI (Master-Out-Slave-In) and SCLK (Serial Clock). MISO (Master In Slave Out) would be for bidirectional SPI communications, which is not the case here. Finally, nearby pin 25 is a ground pin, which will be of use as well.

To connect the LED pixels to the Pi, using the identified wires and pins, put Clock to Clock, Data to Data, and Ground to Ground:

* Green wire --> Pi Pin 23
* Yellow wire --> Pi Pin 21
* Blue wire --> Pi Pin 25
* Red wire --> Not connnected to Pi

<img style="float: right;" src="/images/pixels/hookup.jpg" width="240 px" /> 

As seen in the photograph, this pixel string has a pair of pigtail wires for the DC power, which are connected to the 5V power supply. Again, no +VDC is connected between the Pi and the pixels, but they do share a ground. I apologize that I did not put the effort into selecting header wires that match the pixel wire colors.

# OLA For Real

I've verified that OLA is operational using the ArtNet input and the Dummy output, and I've connected the pixel strip to the Pi, so I'm ready to take the next step and control the pixels over SPI.

<img style="float: right;" src="/images/pixels/ola.png" width="240 px" /> 

First thing is to edit the OLA universe and change the output plugin from Dummy to SPI. There are by default two SPI devices available on the Pi, and I want to select the SPI output identified with `spidev0.0`.

Now I'm ready to write some data to the pixels. For that I'm going to go back to the Node.js container I used earlier.

# Art-Net On Node

OLA includes its own Web API where you can POST DMX data directly to the server for a particular Universe. I have chosen not to use that interface and instead use Art-Net from a different computer on my home network to test/prove end-to-end I/O through OLA. One reason for this is I may integrate some Art-Net functions in a Node.js application already running on my network that would allow [SmartThings][] control and execution of DMX commands.

We saw above that the `artnet` package for Node doesn't need a lot of configuration to get it working: set up an Art-Net connection to the OLA server, and start sending data.

Remember, setting up the Art-Net connection looks like this:

```
var artnet = require('artnet')({host: '192.168.1.XXX'});
```

And sending data looks like this:

```
artnet.set(N,[value1,value2,value3,...,value75])
```

This command only has two arguments: "N" which is the address in the Universe to *start* setting values, and an Array of values to set[^2]. Each value in the array is applied to a slot in the universe starting with the "N" value.

Keep in mind that each LED pixel consumes three slots: one each for its red, green, and blue values[^3]. To set the first pixel in the string to white, issue the command:

```
artnet.set(1,[255,255,255])
```

So that starting at slot 1, set the value 255 three times; the result being that slots 1, 2, and 3 each have the value 255.

Since there are 25 LEDs in my string, there are a total of 75 slots in the universe that I can reasonably set. To turn all the pixels to white, I can set each slot to the value 255:

```
artnet.set(1,Array(75).fill(255));
```

This is using the Javascript Array functions to create an array of length 75 and fill it with the value 255. This is an easy way to set white values for the whole string of pixels. What if I wanted every pixel to be red? The array I need for 75 red pixels is a repetion of the triplet `[255,0,0]` which can be very tedious to create by hand. Instead, I have written a short Javascript function[^4] that takes an RGB value and creates an array of the appropriate length:

```
function monochromePixels( c, r, g, b ) {
  var _r = Array(c).fill(r);
  var _g = Array(c).fill(g);
  var _b = Array(c).fill(b);
  v = [].concat.apply(_r.map(function (e, i) { return [_r[i], _g[i], _b[i]]; }));
  return v.reduce(function(a,b){ return a.concat(b) });
}
```

This function takes a length `c` and a color value as `r`, `g`, and `b` and creates from them the array needed to represent that color value. I can use this function to turn all my pixels red:

```
artnet.set(1,monochromePixels(25, 255, 0, 0));
```

Now, if I wanted a string of LEDs that were all the same color, I could have just bought one and saved myself a lot of hassle. Next time I'll play around with some code that leverage these pixels to do some cool things.

In the poorly-produced GIF below, you can see my pixel strip changing from red to green to off as I send different values through ArtNet using the `monochromePixels` function.

![demo](/images/pixels/demo.gif)

[LED Pixels]: https://www.adafruit.com/product/738
[Open Lighting Architecture]: https://www.openlighting.org
[Raspberry Pi]: https://www.raspberrypi.org
[Docker]: https://www.docker.com/what-docker
[Docker Compose]: https://docs.docker.com/compose/
[Hue]: /the_tools/hue.html
[SPI]: https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus
[DMX]: https://en.wikipedia.org/wiki/DMX512
[Grafana]: {% post_url 2016-12-10-grafana %}
[Node.js]: https://nodejs.org/en/
[SmartThings]: /the_tools/smartthings.html

[^1]: There's a lot of talk on the Internet about LED Pixels, DMX, and SPI with a lean toward performance characteristics, refresh rates, and maximum attainable frames per second. I won't be concerned with these statistics or maths, but they can be found easily through Google.

[^2]: An "Array" is programming-talk for a "list". In JavaScript and many other languages, square brackets (`[` and `]`) are used to identify an Array or List of things. You can also usually access the primitive functions used to create such an object, like the command `Array(x).fill(y)` that creates in the computer's memory an array object of length x and then "fills" it up with the value y. In this exercise, the arrays are always of length (3 * Number of LED Pixels) and the values are always between 0 and 255.

[^3]: Since a DMX Universe has 512 slots, you can have 170 LED pixels in a single universe. At 25 pixels per string, that's just shy of 7 full strips.

[^4]: I'm not much of a JavaScript programmer; the examples here are functional but probably not optimal. I've avoided introducing any dependencies or additional libraries that would probably make some of the things easier, but I wanted to make sure that if you pull Node.js "off the shelf" and start copy-pasting this code, it will work.
