---
title: The ELK Stack
layout: page
tags: [elk-stack]
---
* TOC
{:toc}

The "ELK Stack" is a group of software from [Elastic][1] that works together as a suite to ingest, process, store, index, and visualize data. Its use cases are many, and its cost is zero, so it's a good place to start.

Usually ELK is concerned with consuming log files from systems and applications, but the "E" in ELK, Elasticsearch, is a NoSQL document store and search engine that doesn't care what you put in it.

I don't think I'll be using the L, Logstash, right now, but it's the piece that's concerned with, essentially, ETL operations with log files. It takes log files, cuts them up into records that Elasticsearch can comprehend, and puts them into the database.

The last piece, K for Kibana, is the part that you actually look at. It connects to Elasticsearch and lets you explore the data you've put in there and create visualizations and dashboards from it.

These programs aren't the only gigs in town for these features, and they're not even married to each other. If it doesn't work out, I'll just drop in something else.

# Elasticsearch

First up is [Elasticsearch][4]. It comes with a long bullet-point list of things it's good at, like scaling, distribution, and high availability, none of which I'll use on the Raspberry Pi on which it'll run.

Setup is very straightforward, and I'm going to keep it that way for now. I'm running [Raspbian][2] on my Pi, which is a Raspberry-flavored spin on [Debian][3]. It's quite quick to download the `.deb` package for Elasticsearch and install it with `dkpg -i`. It sets up a user for the Elasticsearch service and puts all the files where they need to go. It's written in Java, so the prerequisites are minimal. Raspbian by default installs so much stuff, including a JDK, that I don't have to think about it.

Before I start Elasticsearch, I hit the `/etc/elasticsearch/elasticsearch.yml` configuration file to open up the network binding to include `_site_` instead of the default of `_local_` so I can access Elasticsearch from other hosts on the network.

Besides some cosmetic settings in this file, I need to set a good path for my data and logs. The Raspbian installer makes an anemic 2GB filesystem for `/`, so the rest of my large SD card is mounted at `/opt/local`. After creating a `data` and a `logs` directory in `/opt/local/elasticsearch` I can note these locations in `elasticsearch.yml`.

Next I edit the `/etc/default/elasticsearch` file, which defines the environment in which Elasticsearch will operate. Because the Raspberry Pi (Version 2) has 1 GB of RAM, and Elasticsearch wants half of your system RAM, I can set the `ES_HEAP_SIZE` environment setting to "512m" here and give the program what it wants. 

Any other configuration tuning steps I can address when or if the need arises.

Finally, I can enable (tell the Elasticsearch service to start with the system at start-up) and start the service with `systemctl enable elasticsearch && systemctl start elasticsearch`.

With Elasticsearch running, there's very little else I need to do to make it work. Even though it's a type of database, I don't need to set up tables or schemas. When I get around to loading in data, it will be with well-structured JSON, and Elasticsearch will just take care of it.

I can make sure Elasticsearch is running with a quick request to its listening port (9200):

```
pi@raspberrypi ~ $ curl http://localhost:9200
{
  "name" : "rpi-1",
  "cluster_name" : "jennings-es",
  "version" : {
    "number" : "2.3.3",
    "build_hash" : "218bdf10790eef486ff2c41a3df5cfa32dadcfde",
    "build_timestamp" : "2016-05-17T15:40:04Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.0"
  },
  "tagline" : "You Know, for Search"
}
```

[1]: http://www.elastic.co "Elastic"
[2]: https://www.raspberrypi.org/downloads/raspbian/ "Raspian"
[3]: https://www.debian.org "Debian"
[4]: https://www.elastic.co/products/elasticsearch "Elasticsearch"

# Kibana

With Elasticsearch up and running, the next step in the stack (since I'm skipping Logstash; this is just a EK stack, I guess. Or E_K.) is to get an instance of Kibana up and running.

Like for Elasticsearch, I'd love to download and use the `.deb` package for [Kibana][5]. Unfortunately, since I made the poor decision to run this stack on a Raspberry Pi, it's not that simple. The Pi uses an ARM processor and the easy-to-digest Kibana packages are built for x86 instruction sets.

This isn't the end of the road. It'll just be a bit more fiddly to get running, but not that bad.

## Kibana and Node.js

Kibana is built atop Node.js, a Javascript application server. So first things first, I need to install Node and download the Kibana source package on my Pi.

Talk about easy. Node.js on a Pi is a solved problem, so I just need to download a `.deb` package for it and install.

```
pi@raspberrypi ~/Downloads $ curl -O http://node-arm.herokuapp.com/node_latest_armhf.deb 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 5485k  100 5485k    0     0  1933k      0  0:00:02  0:00:02 --:--:-- 1933k

pi@raspberrypi ~/Downloads $ sudo dpkg -i ./node_latest_armhf.deb 
Selecting previously unselected package node.
(Reading database ... 99893 files and directories currently installed.)
Preparing to unpack ./node_latest_armhf.deb ...
Unpacking node (4.2.1-1) ...
[...]

pi@raspberrypi ~/Downloads $ node -v
v4.2.1
```

From the Kibana site I download the "Linux 64-bit" tarball and unpack it into `/opt/local`. Then like a good boy I create a softlink between whatever directory it made and the more legible `/opt/local/kibana`. For good measure, I'll `chown` the Kibana directory so that the `elasticsearch` user owns it.

Node is a lot of stuff I don't need to deeply understand right now. It's a server and a platform, a repository and a package manager. It's a lot of things, but all I care about is getting Kibana running, and the next step in doing that is to install the Node package dependencies it requires.

```
cd /opt/local/kibana
sudo npm install
```

Since `npm` is the Node Package Manager, the "install" directive reads some nonsense from the Kibana directory and installs the Node things it needs to run.

If I try to start the Kibana server now, it will fail in a cryptic way, because it's hoping it can use the Node.js binaries with which it was delivered that live in the `./node/bin` directory. But this is a Raspberry Pi and these are not the binaries you're looking for.

The solution is to nix the `node` and `npm` binaries in this folder and replace them with (links to) the real ones we installed when we installed Node.

```
pi@raspberrypi /opt/local/kibana $ cd node/bin/

pi@raspberrypi /opt/local/kibana/node/bin $ ls -altrh
total 24M
-rwxrwxr-x 1 elasticsearch elasticsearch  24M May  6 00:38 node
-rwxrwxr-x 1 elasticsearch elasticsearch 1.9K May  6 00:38 npm
drwxrwxr-x 6 elasticsearch elasticsearch 4.0K Jun 18 15:32 ..
drwxrwxr-x 2 elasticsearch elasticsearch 4.0K Jun 18 15:32 .

pi@raspberrypi /opt/local/kibana/node/bin $ file ./node
./node: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.9, BuildID[sha1]=d3181fdf438c833e8cf8ffaa1023bd9e27636db4, not stripped

pi@raspberrypi /opt/local/kibana/node/bin $ file `which node`
/usr/local/bin/node: ELF 32-bit LSB executable, ARM, EABI5 version 1 (GNU/Linux), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 2.6.32, BuildID[sha1]=1944b8fa8826d8b8d990b63eab4732661f3a8d93, stripped

pi@raspberrypi /opt/local/kibana/node/bin $ sudo mv node node~
pi@raspberrypi /opt/local/kibana/node/bin $ sudo mv npm npm~
pi@raspberrypi /opt/local/kibana/node/bin $ sudo ln -s `which node` node
pi@raspberrypi /opt/local/kibana/node/bin $ sudo ln -s `which npm` npm

pi@raspberrypi /opt/local/kibana/node/bin $ ls -l
total 24300
lrwxrwxrwx 1 root          root                19 Jun 18 15:45 node -> /usr/local/bin/node
-rwxrwxr-x 1 elasticsearch elasticsearch 24879040 May  6 00:38 node~
lrwxrwxrwx 1 root          root                18 Jun 18 15:45 npm -> /usr/local/bin/npm
-rwxrwxr-x 1 elasticsearch elasticsearch     1928 May  6 00:38 npm~
```

Now I can start Kibana with `/opt/local/kibana/bin/kibana` and it'll go! Then I can visit my Pi at port 5601 and see Kibana answer.

Well, Kibana doesn't so much "answer" as it "asks" some bootstrappy kinds of questions like "What Elastic search index" to run search and analytics against. How should I know? It's fine, by default it suggests "logstash-*" which is the baked-in assumption that this is an ELK stack instead of an E_K stack. I can change this later when I know what my data looks like.

## Kibana and Systemd

The last Kibana problem is that back in my ssh session to my Pi, Kibana is actually running in the foreground. Using the `systemd` init system in Raspian, I can do better than this.

It's really easy to set up a new `systemd` service (or unit). I'm sure had I been able to just use a package to install Kibana, it would have taken care of it in one way or another.

I'm going to create a file named `/etc/systemd/system/kibana.service` with the following content:

```
[Unit]
Description=Kibana 4
After=elasticsearch.service

[Service]
Type=simple
User=elasticsearch
EnvironmentFile=/etc/default/kibana
ExecStart=/opt/local/kibana/bin/kibana

[Install]
WantedBy=multi-user.target
```

And a file at `/etc/default/kibana` with:

```
CONFIG_PATH=/opt/local/kibana/config/kibana.yml
NODE_ENV=production
```

Then I invoke the `systemd` commands to put all this in action:

```
systemctl daemon-reload
systemctl enable kibana
systemctl start kibana
systemctl status kibana
```

This process reads new or changed service files, like the one I wrote for Kibana, enables Kibana for startup, starts Kibana, and then shows me the status. If everything looks good I can go back to my browser and refresh my page and make sure it's running.

There are a bunch of configuration directives in the `/opt/local/kibana/config/kibana.yml` file but nothing that needs to be adjusted right now. I could specify a PID file and make the `systemd` unit file aware of it, or I could specify a location for Kibana's log file (it defaults to logging only to STDOUT), but otherwise the out-of-box defaults are fine for a casual E_K stack like this one.

[5]: https://www.elastic.co/products/kibana

# Pages About the ELK Stack

<ul>
{% assign sorted = site.elk_stack | sort: 'moddate' | reverse %}
{% for item in sorted %}
  <li>
    <a href="{{ item.url }}">{{ item.title }}</a>
    <span class="date">{{ item.moddate | date: "%B %-d, %Y"  }}</span>
  </li>
{% endfor %}
</ul>

