---
title: Raspberry Pi Serial Server
layout: page
tags: the_stereo raspberry_pi
moddate: 2016-08-03
---

I have an old Raspberry Pi sitting around. It's one of the first or second edition Pis, the kind that uses a full-size SD card and doesn't even have any built-in Wi-Fi. I believe it is a [Raspberry Pi 1 Model B](https://www.raspberrypi.org/products/model-b/). It's still a capable little computer even if it's not very fast or doesn't have much RAM.

I don't remember what I had been doing with it in the past, and it has an 8GB card in it, so I just blew that away and put on the latest Raspbian Lite image from May 2016. After booting it up, I did the usual first-step stuff like `apt-get upgrade` and making sure the SD card was being used to its full extent; it was.

My goal for this little guy is to attach a USB->Serial adapter and get it talking to my [Monoprice 6-Channel Multi-Zone Whole-House Amplifier](http://www.monoprice.com/product?c_id=109&cp_id=10918&cs_id=1091801&p_id=10761&seq=1&format=2). Or, as I'll refer to it from now on, "The Stereo."

# The Stereo

In the pre-history days before I bought this thing, and in a few of the months since, I've looked around the web for projects that make use of the serial port. I've seen Windows-only apps and weird PHP interfaces, but nothing I felt like I'd want to hang my hat on.

Recently I found [this project][], a Node-based API server for this stereo written in less than 200 lines of code. Not too shabby. Sure, that doesn't include the dependencies, but come on, nothing else I've seen is as short and sweet as this.

I forked the Github repository and cloned it onto my RPi at `/opt/local` and since the name is so inscrutable ("mpr-6zhmaut-api") and hard to type, I linked it to `/opt/local/stereo-server`.

## Node

I downloaded a version of Node for the Pi from [Node-Arm](http://node-arm.herokuapp.com/), which is easy to download and install since it's a `.deb` package.

A lot of little projects are popping up that use Node as their basis. Say what you will about JavaScript, and you might be right, Node is a very capable and agile application server with an impressively deep ecosystem.

To better support that ecosystem, after I installed Node I did a "global" install of the `node-gyp` package which is a tool that builds Node addons, and I figure I'm going to need it. It takes a little bit of time to do its work:

```bash
sudo npm install -g node-gyp
```

Once that's done, change to the software directory and install any Node requirements for this application (I had to install the serialport dependency directly with the `--build-from-source` flag to avoid weird build errors that aren't worth troubleshooting):

```bash
cd /opt/local/stereo-server
npm install
npm install serialport --build-from-source
```

This project doesn't require a great deal of add-ons, but it does use `express` which appears to be a fully-featured [application server](http://expressjs.com) and `serialport` which is kind of important for a project that's going to use a serial port to communicate with a device.

# Serial Killer

Ever since the dawn of time, man has strove to destroy the Serial Port. Some of my best days as a kid were spent chanting "COM1 3F8 COM2 2F8 COM3 3E8 COM4 2E8" so that I would always know the I/O addresses of the 4 serial ports in my computer, should I need one for a modem, or a joystick, or whatever. It used to be that anything that wasn't a printer connected to a serial port.

Then USB came along, and every computer got rid of the serial port (and the parallel port, too, making my old Zip drive obsolete), but someone forgot to tell the hardware designers because networking gear and certain mid-to-high-end stereo systems still use serial ports for connectivity and communication.

The solution was to make and market a new peripheral, the USB-to-Serial adapter. I have a [Trendnet TU-S9](http://amzn.to/29CQw9I), which according to Amazon I purchased in 2012[^1].

There are a lot of companies selling these things, but there's really only a handful of different chipsets driving them. Problem is, you don't always know which chipset is going to work on your platform or what chipset is in the thing you're thinking of buying.

The Trendnet TU-S9 has a "PL2303" chipset in it, which according to this [Verified Peripherals](http://elinux.org/RPi_VerifiedPeripherals#USB_UART_and_USB_to_Serial_.28RS-232.29_adapters) site, "works fine" on the Raspberry Pi.

One quick way to tell what chipset your adapter is using is to plug it in and take a look at it:

```
pi@raspberrypi:/opt/local/stereo-server $ sudo lsusb
Bus 001 Device 004: ID 067b:2303 Prolific Technology, Inc. PL2303 Serial Port
```

The computer doesn't care what brand is stamped on the plastic case holding the electronics, it only sees the chipset. 

And indeed, at first glance I would say that this "works fine" on the Raspberry Pi, since with the adapter plugged in I can see the device `ttyUSB0` in `/dev/`, which is a fancy way of saying the interface/file by which Linux applications will access the USB serial port is present and accounted for.

That's the hard part handled; my Raspberry Pi has a serial port[^2]. Now I just need a serial cable to connect it to The Stereo.

[this project]: https://github.com/jnewland/mpr-6zhmaut-api

# Configuring the Stereo Server

At this point I have my serial port available, and I have the application server software installed, so I'll take a quick look at what needs to be configured to make sure one works with the other.

Inside the Node application, there are very few things to look at. Really, the whole thing is contained in `package.json` which tells Node what the requirements are, and `app.js` which is the actual less-than-200-lines of code that make the things happen. I'll look at the latter file.

I see an interesting part of the code:

```javascript
var SerialPort = serialport.SerialPort;
var device     = process.env.DEVICE || "/dev/ttyUSB0";
var connection = new SerialPort(device, {
  baudrate: 9600,
  parser: serialport.parsers.readline("\n")
});
```

According to this, I shouldn't have to do anything in particular. I already noted that my serial port is available at `/dev/ttyUSB0` and that is the device this application will use by default. If this weren't the case, it looks like I could use a `$DEVICE` environment variable to specify a different one. Finally, the default baud rate of 9600 looks correct.

Another interesting part of the code is a single line way at the bottom:

```javascript
  app.listen(process.env.PORT || 8181);
```

It looks like the application will by default listen on port 8181 but I can change this with the `$PORT` environment variable, which I shouldn't need to do.

In order to start this thing up, I'll execute `npm start` just to make sure everything looks like it's operational, i.e., no errors are delivered to the console. This is the case, but I don't want this thing running in the foreground. I'll write a `systemd` unit file for it at `/etc/systemd/system/stereo-server.service`:

```
[Unit]
Description=Stereo Serial Server
Requires=network.target
After=dhcpcd.service

[Service]
Type=simple
User=nobody
Group=dialout
EnvironmentFile=/etc/default/stereo-server
WorkingDirectory=/opt/local/stereo-server
ExecStart=/usr/local/bin/node /opt/local/stereo-server/app.js

[Install]
WantedBy=multi-user.target
```

This application doesn't need a specific user account ("nobody") but should have permissions of the "dialout" group since that group owns the `/dev/ttyUSB0` device. I have to make the `EnvironmentFile` this unit is referencing, too:

```
sudo touch /etc/default/stereo-server
echo PORT=8181 | sudo tee -a /etc/default/stereo-server
echo DEVICE=/dev/ttyUSB0 | sudo tee -a /etc/default/stereo-server
echo NODE_ENV=production | sudo tee -a /etc/default/stereo-server
```

Here I did not have to, but I specified the `PORT` and `DEVICE` environment anyway. 

Now to make the service active, I'll issue a few `systemctl` commands:

```bash
systemctl daemon-reload
systemctl enable stereo-server
systemctl start stereo-server
systemctl status stereo-server
```

That's it. Time to hook this thing up and see if it works.

# Serial Ports

I talked a little bit about serial ports already, but even though what comes next is "hook up a cable," there's more to it than that.

In the world of serial ports, which is an old, old world, there are two kinds of devices: DTE and DCE. The DTE (Data Terminal Equipment) device is supposed to be a computer or terminal; something that initiates connections and does something with the data that that connection returns. A DCE (Data Circuit-terminating Device) is supposed to be a peripheral device like a modem, a serial mouse, a disk drive, a diagnostic port on some equipment, that sort of thing.

Computing is littered with these kinds of tuples: DTE/DCE; Client/Server; even the generic Tx/Rx. It's all about when one device is talking to another, and serial ports are one of the oldest ways this could be accomplished.

There are two kinds of 9-pin serial ports: male and female. Conventionally, a DTE device has a male serial port on it, and a DCE device has a female. You would use a standard serial cable to connect these guys together, like a computer and a modem, and get them talking. Alternately, if you wanted to connect two DTE devices, such as to support multiplayer gaming from 1989, you would use not just a cable that has two female ends, but that cable would also have to be "null modem," which is to say the send/receive wires are internally swapped. As far as I know there's no use case where you'd connect two DCE devices together.

I say all this as preface to the fact that the Monoprice amplifier has a female DB9 port on it, which according to convention makes it a DCE device, so I want to use an ordinary "straight-through" [serial cable](http://amzn.to/2a5zQW2) instead of a [null modem](http://amzn.to/2anGMSM) serial cable.

Given the facts laid out above, you can define at least 6 different serial cables: M/F straight-through (standard); M/M straight-through; F/F straight-through; M/F null-modem; M/M null-modem; and F/F null modem (standard). Indeed, if you go shopping, you can find all these varieties on sale. You can also find "gender changers" that just pop on the end of a cable you already have to change its disposition; these are also available in straight-through (normal) and null-modem (effectively making the whole cable a null modem cable) varieties. So if you wanted to cover all your bases, you'll have to have a bunch of different pieces of kit on hand.

If this sounds ridiculous, don't judge before considering the state of USB: You've got USB type A (the standard big block that goes into a computer), type B (smaller squarish block that goes into a peripheral), mini, micro (which plug into various portable devices), and now the vaunted USB-C. Amazingly, there are [even more than that](http://www.cablestogo.com/learning/connector-guides/usb), so serial ports don't have a monopoly on stupid cable configurations, which you probably know if you've ever wanted to plug your phone in at a friend's house.

# Hooking up the Amplifier

So the amp has a female serial port on it, my Pi has a male. As noted, the conventional approach would be to use a straight-through M/F cable. 

With it hooked up, I should be able to talk to it, so I'll do the basic thing and use the server I installed and see if it works:

```
pi@piserver1:~ $ curl -XGET http://localhost:8181/zones
[{"zone":"11","pa":"00","pr":"00","mu":"00","dt":"00","vo":"15","tr":"07","bs":"07","bl":"10","ch":"06","ls":"01"},{"zone":"12","pa":"00","pr":"00","mu":"00","dt":"00","vo":"20","tr":"07","bs":"07","bl":"10","ch":"06","ls":"01"},{"zone":"13","pa":"00","pr":"00","mu":"00","dt":"00","vo":"13","tr":"07","bs":"07","bl":"10","ch":"06","ls":"01"},{"zone":"14","pa":"00","pr":"00","mu":"00","dt":"00","vo":"25","tr":"07","bs":"07","bl":"10","ch":"01","ls":"01"},{"zone":"15","pa":"00","pr":"00","mu":"00","dt":"00","vo":"25","tr":"07","bs":"07","bl":"10","ch":"01","ls":"00"},{"zone":"16","pa":"00","pr":"00","mu":"00","dt":"00","vo":"25","tr":"07","bs":"07","bl":"10","ch":"01","ls":"00"}]
```

That's a lot of output, but it means it's working. Each of the 6 zones on the amp are reporting in with a chunk of JSON, for example:

```json
{"zone":"11","pa":"00","pr":"00","mu":"00","dt":"00","vo":"15","tr":"07","bs":"07","bl":"10","ch":"06","ls":"01"}
```

The above is for Zone 1 (it says "11" because it's zone 1 on unit 1, and you can stack 3 of these things for 18 total zones). The guy who wrote the app decided to keep very short tokens to represent the different configuration details for this thing, but in order the amp is reporting the "Public Address Status", "Power Status", "Mute", "Do Not Disturb", "Volume", "Treble", "Bass", "Balance", "Source Channel", and "Keypad Status".

This is success, but it's not very useful yet. It's an important building block, though: a REST API (or something close enough the distinction doesn't matter) to control my stereo through its serial interface.

Next step: Integrate this building block with [SmartThings](/the_tools/smartthings) and get this thing participating in my Smart Home activities.

Continue to [Part 2]({% link collections/_raspberry_pi/rpi-serial-server-2.md %}).

[^1]: Turns out after fighting with this thing for a while, it's broken. You can always tell if a serial port is working through a loopback test, and this thing failed every attempt I made. I ended up buying a [new PL2303-based adapter](http://amzn.to/2alDYBo) which worked great right off the bat.

[^2]: The Raspberry Pi also has a built-in serial port as part of its GPIO set of pins, and this is available to the system at `/dev/ttyAMA0` but for simplicity's sake I've gone the USB route. The GPIO serial port on the Pi runs at 3.3v instead of RS232's 5V which means you probably need [additional kit](http://amzn.to/2alEXlc) to make it work anyway.
