---
layout: page
title: The Dashboard (Part II) - Logstash
tag: elk-stack
moddate: 2016-06-24
---
I was wondering if I'd have a use for [Logstash][] in my [ELK Stack][]. So far I've used Elasticsearch (the "E") and Kibana (the "K") in setting up a [dashboard][] for my home. It turns out there is a reason to use Logstash (the "L") after all.

# Bulk Data

While Elasticsearch does have a bulk-import API, it's not a good fit for what I want to do. If I had large amounts of JSON-formatted documents ready to pipe directly into an index, sure, I'd use the bulk API. Instead, I have a lot of CSV data I want to make available in Elasticsearch, and Logstash will be the right tool for the job.

Logstash's role in the ELK stack is to take arbitrary data, such as its namesake, a log file, and parse it into a format that Elasticsearch will be happy to accept. In this case, I'm going to use it to process CSV files in bulk.

There are two sets of CSV data I want to add to my Elasticsearch database. The first is a set of rainfall readings from a weather station near my house. The second is a daily record of energy use downloaded from my electric company. I'll actually focus on the second of these today.

# Installing Logstash

I'm going to treat Logstash a little differently. I don't need it running all the time, so I'm going to use it as an on-demand data pusher, at least for now. Regardless, the install process is straightforward. Like Elasticsearch, I need only download the `.deb` package file from the Logstash website and use `dpkg -i` to install it on my Raspberry Pi.

Once the `.deb` package is installed, I'll configure it so it won't start automatically with the system, but I'll be able to start and stop it at will when I have data to process. In a production environment you might have full-time Logstash instances listening for and parsing messages from a number of locations around the clock, but my CSV files are a one-and-done process.

I'll edit the `/etc/defaults/logstash` file to get a sense of the kind of environment Logstash needs to run, and I'll set the `LS_HEAP_SIZE` to `100m` so memory doesn't get out of hand.

## Fixing JRuby

Logstash uses an embedded version of JRuby that doesn't work on the Raspberry Pi. I really shouldn't have picked this platform (I could have run Logstash from anywhere). The problem arises from a library called `jffi` or "Java Foreign Function Interface".

I can rebuild it. I have the technology.

I'll grab the latest `jffi` source code from Github, build it, and replace the `libjffi-1.2.so` library with my newly built one. I'll need a couple of tools to make this happen, so I'll install `zip` and `ant` in the usual way.


```bash
sudo mkdir -p /opt/local/src
cd /opt/local/src
sudo git clone https://github.com/jnr/jffi.git
sudo apt-get install ant zip
```

Now I'll build the copy of `jffi` here on the Pi and replace the library file in my Logstash installation.

```bash
cd jffi
sudo ant jar
sudo mv /opt/logstash/vendor/jruby/lib/jni/arm-Linux/libjffi-1.2.so /opt/logstash/vendor/jruby/lib/jni/arm-Linux/libjffi-1.2.so~
sudo cp build/jni/libjffi-1.2.so /opt/logstash/vendor/jruby/lib/jni/arm-Linux/libjffi-1.2.so
```

With the updated library in place, I can test Logstash by running it with a no-nonsense inline configuration that reads from `STDIN` and writes to `STDOUT`.

```
USE_JRUBY=1 LS_HEAP_SIZE=64m /opt/logstash/bin/logstash -e 'input { stdin { } } output { stdout { } }'
```

Once it starts up, I can type "test" and see "test" parroted back at me.

# Configuring Logstash

Logstash needs a configuration file to work. This file has three main sections in it, `input`, `filter`, and `output`.

I'll create a file in my `pi` user's home directory called `logstash-csv.conf` and edit it so it has this structure:

```
input {
}

filter {
}

output {
}
```

One at a time, I'll consider and configure these three sections.

## Input

An input section tells Logstash what it's taking in and from where. My goal is to load some CSV files from a local filesystem location. Presumably I will copy these files to my Pi so they're available. Let's say I'll put them in `/tmp/logstash-csv` because I don't really care about what happens to them when I'm finished. This is expressed with the `path` configuration directive.

The complete documentation for Logstash's `file` input plug-in is available [here][]. Of all the configuration directives available for it, another one I need is `start_position` because my files are old data, not new streams. I'll also use `sincedb_path` and point it to `/dev/null` so I can easily make Logstash read the same file over and over again whenever I want. Finally, I'll use `ignore_older` because for some reason Logstash is treating even brand new files as old, maybe because my filesystems are mounted with `noatime`, maybe for some other reason.

My complete input section looks like this after adding an empty `stdin` input, with the `file` section commented out for now so I can test the pipeline:

```
input{
	stdin { }
#	file {
#		path => "/tmp/logstash-csv/*.csv"
#		start_position => "beginning"
#		sincedb_path => "/dev/null"
#		ignore_older => 0
#	}
}
```

## Output

I'm going to skip to the end and configure Logstash's output. Like the input, there are a lot of options; Logstash can output and thus talk to a wide array of message brokers, which gives it utility outside an ELK stack.

For now, I'm going to set up two output plugins, `elasticsearch` and `stdout`. That first one should make sense, and the second one is so I can review Logstash's work as it does it.

First up is `elasticsearch`. According to the [documentation](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html), I don't actually have to configure anything if the defaults are suitable.

I'll set up my output configuration block for the Elasticsearch plug-in like this:

```
elasticsearch{
	action => "index"
	index => ""
	document_type => ""
}
```

The two empty directives, `index` and `document_type`, require some thought. As mentioned above, the CSV data is going to be a daily record of my electric consumption according to my utility company. It follows that it might be appropriate to use the same index "utility" that I created when I first loaded some utility bill documents to my database.

The second directive, `document_type` will differentiate these readings from those other utility bill documents. I used a type called "bill" for those, so I can use a type called "usage" for these records.

By default, the plug-in will assume that Elasticsearch is running on `localhost`, which is correct, so I have not specified a `hosts` directive in my configuration.

Next up is the `stdout` plugin. Nothing much to it. I will specify the output codec to be "rubydebug" which is a nicely formatted log slug.

My complete output section looks like this:

```
output {
#	elasticsearch{
#		action => "index"
#		index => "utility"
#		document_type => "usage"
#	}
	
	stdout {
		codec => "rubydebug"
	}
}
```

It seems odd to have commented out the entire `elasticsearch` section but I've done so in order to vet the output before actually sending anything over to Elasticsearch.

# The Data

Before I configure the `filter` section of my Logstash configuration, I need to understand completely the format of the CSV data I'm going to load.

I've downloaded the daily usage available from my power company as a CSV file. Opening it, I see that it has extra detail at the top of the file, specifically 3 extra lines, then the field headers, and finally the data records. It looks like this:

```
Account XXXXXXXXXX Address 123 N MAIN ST
Meter XXXXXXXX
06/22/2016--10:37
Day,Date,Meter Read,Total kWh,Max Temp,Min Temp,Avg Temp,Est Cost

Mon, 06/20/16, 13254, 57, 95, 72, 84, 8.436
Sun, 06/19/16, 13197, 44, 91, 73, 82, 6.512
Sat, 06/18/16, 13153, 74, 89, 73, 81, 10.952
Fri, 06/17/16, 13079, 75, 93, 78, 86, 11.1
Thu, 06/16/16, 13004, 84, 96, 75, 86, 12.432
```

Right off the bat I know I can axe the first 5 lines of the file, including the column headers and the blank line following. I'll create a copy of the file and make that edit, then copy the edited file to my Pi at the location I noted under Input.

I see there are 8 fields in the file, and I suspect the first one won't be very valuable. I probably don't need the "Meter Read" field, either, since all I really care about is the delta between readings.

There are also some pieces of metadata that describe the fields but aren't present, such as the fact that the units are in kWh, the temperatures are in Fahrenheit, and the cost is in USD.

I notice that there is some whitespace between the fields on each row. If I leave that alone, Logstash will interpret those fields as strings, so I need to either find/replace whitespace with null or work around that in my filter.

Finally, I'll `tail` the file and make sure there aren't any blank lines at the end, and if there are, I'll edit the file to remove them.

Before moving on, I'll review the [documentation](https://www.elastic.co/guide/en/logstash/current/plugins-filters-csv.html) for the CSV filter, taking particular note of the `add_field`, `convert`, `columns`, and `remove_field` options.

# The Filter

The filter is what Logstash does with a data record ("event") in between having ingested it as input and emitting it as output. Again, there are myriad options, and since this is CSV data, I'll use the `csv` filter to process it.

Taking what I've learned from looking at the data file and the Logstash documentation, I'll create a configuration section that looks like this:

```
filter {
	csv {
		columns => ["day","@timestamp","meter_read","usage","temp_max","temp_min",
				    "temp_avg", "cost"]
		add_field => {
			"usage_units" => "kWh"
			"temp_units" => "F"
			"cost_units" => "USD"
			"utility_type" => "electric"
		}
		remove_field => ["day","meter_read"]
		convert => {
			"@timestamp" => "date"
			"usage" => "integer"
			"temp_max" => "integer"
			"temp_min" => "integer"
			"temp_avg" => "integer"
			"cost" => "float"
		}
	}
}
```

Here I'm specifying column names, adding three metadata fields, and removing the two fields I don't want. I've identified the "Date" column as `@timestamp` so the event will use that value as its origin instead of the actual time that Logstash processes the event. I've also specified a conversion for the fields that need it.

With my configuration complete, I'll make sure it's in good shape by testing it with logstash:

```bash
/opt/logstash/bin/logstash -t -f /home/pi/logstash-csv.conf 
```

I should get back "Configuration OK" so I'm ready to rock and roll. I'll repeat the above command but omit the `-t` flag. That will run Logstash for real with my config file. Once I get a "Pipeline main started" I can test my workflow by pasting a line from my CSV file into STDIN.

The result is close, but no cigar:

```
Pipeline main started
Mon, 06/20/16, 13254, 57, 95, 72, 84, 8.436
{
        "message" => "Mon, 06/20/16, 13254, 57, 95, 72, 84, 8.436",
       "@version" => "1",
     "@timestamp" => " 06/20/16",
           "host" => "raspberrypi",
          "usage" => 57,
       "temp_max" => 95,
       "temp_min" => 72,
       "temp_avg" => 84,
           "cost" => 8.436,
    "usage_units" => "kWh",
     "temp_units" => "F",
     "cost_units" => "USD"
}
```

The timestamp has not been turned into a real date. I'll add a `date` filter to my config to handle the conversion, and I'll specify the format with and without the leading whitespace:

```
filter {
	csv { ... }
	date {
		match => [ "@timestamp", " MM/dd/YY", "MM/dd/YY"]
	}
}
```

Now if I rerun Logstash and paste in a record again, the `@timestamp` field is properly identified as such.

# Changing The Event Structure

My event is shaping up, but all the fields are at the root, and I want the document structure to look more like this:

```
{
	"@timestamp": @timestamp,
	"utility": utility_type,
	"usage":
		{
			"value": usage,
			"units": usage_units
		}
	"temperature":
		{
			"min": temp_min,
			"avg": temp_avg,
			"max": temp_max,
			"units": temp_units
		}
	"cost":
		{
			"utility": cost,
			"units": cost_units
		}
}
```

In addition to adding structure to the fields, I've attempted to match the field names in this document to the field names I used to define my utility bills [earlier][dashboard]. To accomplish this I need to move some things around in the Logstash event I've parsed out by using the `mutate` filter:

```
filter {
	csv { ... }
	date { ... }
	mutate {
		add_field => { "utility" => "electric" }
		rename => { "temp_min" => "[temperature][min]" }
		rename => { "temp_avg" => "[temperature][avg]" }
		rename => { "temp_max" => "[temperature][max]" }
		rename => { "temp_units" => "[temperature][units]" }
		rename => { "usage" => "[usage][value]" }
		rename => { "usage_units" => "[usage][units]" }
		rename => { "cost" => "[cost][utility]" }
		rename => { "cost_units" => "[cost][units]" }
		remove_field => ["path","host"]
	}
}
```

# On To Elasticsearch

That should do it. I'll uncomment both the `file` and `elasticsearch` sections of "input" and "output", respectively, and fire up Logstash one last time. It will read my input file and blast the contents to both STDOUT (so I can see what's going on) and into Elasticsearch's "utility" index as the "usage" document type. From there I'm ready to fire up a Kibana dashboard and start looking at my data.
		


[here]: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-file.html
[Logstash]: https://www.elastic.co/products/logstash
[ELK Stack]: /the_tools/elk-stack.html
[dashboard]: {% link _elk_stack/The-Dashboard.md %}
