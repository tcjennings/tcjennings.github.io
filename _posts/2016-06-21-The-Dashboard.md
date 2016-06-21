---
layout: post
title: The Dashboard (Part I)
---

An important component to a smart home is tracking the sorts of things that the house is trying to tell you and putting those facts in context with things you already know.

One of [the tools][] I'm using is an [ELK Stack][], which will let me develop a dashboard on top of documents or metrics that I store in a database. The plan is to read values from other tools implemented in the house and see what's going on. I can store anything I want in the database, as long as it's formatted as JSON.

Right now I have my E_K stack running on a Raspberry Pi and there's no data in it yet. I'm going to create an index (think of it as a database in traditional RDBMS jargon) to hold some documents that represent my utility bills. I already have a spreadsheet where I'm tracking the salient details from my monthly bills, so I will use those columns as a guide for setting up the JSON to describe those bills as documents in Elasticsearch.

I don't have to do anything special to create a new index in Elasticsearch or to pre-define my document structure (though I certainly *can*), so I'll start by brainstorming how the JSON will be structured.

```json
{
}
```

That's an empty JSON document; not very interesting, but you have to start somewhere. I'll add some structure to it.

```json
{
	"date": date,
	"days": days,
	"utility": utility,
	"usage": usage,
	"cost": cost,
	"total": total
}
```

That's pretty straightforward and represents the interesting details from each bill: the bill date, the number of days of service being billed, how much of what utility I used, and what it cost (plus the total bill amount which would include service fees and taxes which I don't need to track separately).

Since this is a one-size fits all format and I have at least three different kinds of utility bills, I can refine this model a little further.

```json
{
	"date": date,
	"days": days,
	"utility": utility,
	"usage":
		{
			"value": scalar,
			"units": units
		},
	"cost":
		{
			"utility": value,
			"total": value
		}
}
```

Here I've broken "usage" into a separate object with "value" and "units" keys, where "value" would be the scalar value of a consumed utility, and "units" is how that value is measured. I'll populate this JSON with details from my most recent gas bill.

```json
{
	"date": "2016-06-13",
	"days": 30,
	"utility": "gas",
	"usage":
		{
			"value": 1,
			"units": "ccf"
		},
	"cost":
		{
			"utility": 0.43,
			"total": 38.53
		}
}
```

That's pretty sickening, but I'll try not to think about it too much. Since it's summer, that gas usage is entirely consumed by the gas cooktop in the kitchen.

That looks pretty good, so I'll try to poke it into Elasticsearch. I'll use the `curl` command to do so.

```bash
pi@raspberrypi ~ $ curl -XPOST 'http://localhost:9200/utility/bill' -d @/tmp/bill.json
{"_index":"utility","_type":"bill","_id":"AVV0A0OuyjOs9zgNdULA","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
```

In this command I've pasted the JSON into a file at `/tmp/bill.json` to make the command consise. The return value includes the `_id` assigned to my document.

I can check my work with the Elasticsearch API:

```
pi@raspberrypi ~ $ curl 'localhost:9200/_cat/indices?v'
health status index   pri rep docs.count docs.deleted store.size pri.store.size 
yellow open   utility   5   1          1            0      5.3kb          5.3kb 
```

There I can verify my "utility" index was created and has a single document stored in it (`docs.count`). You can't tell from here, but the "type" of document I specified ("bill") is associated with that document, and Elasticsearch automatically created a "mapping" (or "schema") for that type of document. I'm concerned with how it interpreted the "date" field, so I'll issue a command to retrieve the mapping for the "bill" type in the "utility" index.

```bash
curl -XGET 'http://localhost:9200/utility/_mapping/bill?pretty'
```

This prints out a lot of stuff, but this is what I'm looking for:

```json
"date" : {
	"type" : "date",
	"format" : "strict_date_optional_time||epoch_millis"
```

Indeed, because I used a well-formatted string for my date ("YYYY-MM-DD"), Elasticsearch successfully interpreted it as a date object instead of a string.

I repeat the process for the last few months of gas bills to get some historical context loaded up.

## The Copa Kibana

With some data in Elasticsearch, I can turn my attention to Kibana, the dashboarding and data exploration component in the ELK stack. Previously unconfigured, when I visit the Kibana URL I first have to tell it an index to look at. That would be "utility", and after a moment Kibana asks me what to use for a date field. The choice is "date" because that's the only date field in the mapping.

Kibana then gives me a look at all the fields in the "utility" index. Most of them are familiar, because they're part of the JSON documents I added to Elasticsearch (like "utility" and "usage.value"); others are unfamiliar metadata fields that Elasticsearch added (like "\_id" and "\_source").

There's a lot I can do here, but I'm going to leave it be for now. The important thing is that every field in my JSON document is present and "indexed".

Next time I'll set up a dashboard in Kibana to look at my data.

[the tools]: /the_tools
[ELK Stack]: /the_tools/elk-stack.html