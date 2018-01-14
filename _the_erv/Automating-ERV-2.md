---
title: Automating the ERV Part 2
layout: page
tags: erv smartthings
---

In [Part 1][] I leveraged the [Z-Wave functionality][] I'd added to my [ERV][] to create some basic automation for its operation. I implemented a [SmartThings][] routine to begin high-speed ventilation on demand and, not liking the open-endedness of that, wrote a Smart App to shut the thing down again after some amount of time.

The next phase of automating the ERV is to implement some "20 minutes on, 40 minutes off" functionality, which will emulate a feature present in a programmable wall control for the unit, which I don't have and don't want.

To throw some complication into the matter, I want this routine to be a little smarter. It should:

* Support any length of time between 1 and 60 minutes.
* Be aware of outdoor conditions and operate only when it is pleasant outside.
* Execute continuously, not in response to executing a routine or flipping a switch.
* Operate at low speed.

I'll start with the SmartApp I have already written and extend it to support this new functionality. For reference, that app is in my [Github repository][].

The first thing I need to do is change the way the SmartApp interacts with my ERV switches. Previously, all I wanted to do was turn on high-speed ventilation for some length of time, after which everything should be shut off. I have two discrete switches, one for "on/off" and one for "high/low", and it was fine to treat them as a group. But now for continuous periodic operation, I don't want it to be high-speed all the time. High speed ventilation is fine for on-demand timer operation, but not for continuous "N minutes of every hour" operation.

# Splitting the Switches

My SmartApp has a preferences section in which I define the devices it will interact with. It currently looks like this:

```groovy
preferences {
	section("ERV Switches") {
		input "switches", "capability.switch", 
		title: "Which ERV switches?", multiple: true
	}
	
	section("Timer Duration") {
		input "minutes", "number", required: true, title: "How Long (minutes)?"
	}
}
```

I'm going to change this section somewhat; I want to differentiate between the two switches, and I want a separate duration value to apply to the periodic ventilation rule I'm working on.

```groovy
preferences {
	section("ERV Switches") {
		input "erv", "capability.switch", 
		title: "Which ERV On/Off switch?", multiple: false
		
		input "hilo", "capability.switch",
		title: "Which ERV Low/High switch?", multiple:false
	}
	
	section("Routine Timer Duration") {
		input "minutes", "number", required: true, 
			title: "How Long (minutes)?"
	}

	section("Periodic Timer Duration") {
		input "pminutes", "number", required: true, 
			title: "How Long (minutes < 60)?"
	}
}
```

Now I will have discrete access to the switches as `erv` and `hilo` throughout the code, and I'll have two separate timer variables, the first of which will, as before, control how long the ERV will run when triggered from a Routine, and the second will be how many minutes of each hour the periodic ventilation will run.

Before I move on, I want to add another preference. One of my requirements is that the periodic ventilation not operate when the outdoor conditions are not "pleasant". That can be a pretty open-ended constraint -- what does it mean that it is "pleasant" outside?

# The Pleasantness Heuristic

The ERV is an *energy recovery* ventilator; it's designed to retain heat energy in the heating season and keep that same heat energy out in the cooling season, so my "pleasantness" heuristic doesn't need to be based (entirely) on outdoor air temperature.

The ERV is also designed to retain/reject *latent* heat, i.e., humidity, but in practice it's not very good at it. In the humid summers, it can and does dump damp air into the house, which makes things indoors much less pleasant. So clearly outdoor humidity should be part of my "pleasantness" heuristic.

Humidity alone isn't a valuable metric. It can be 65F with 80% humidity and we'd call that "pleasant" but the same humidity with an increase of 15F would be unbearably sticky. Luckily, this is a solved problem in meteorology, and the solution is the [Dew Point][]. 

I'll worry about measuring the dew point later, and for now I'll just put in the `preferences` a picker for the outdoor sensor I'll use to do it and a specification for a dew point threshold:

```groovy
preferences {
	{...}
	
	section("Outdoor Conditions") {
		input "outdoor", "capability.relativeHumidityMeasurement",
			required: true, title: "Which Outdoor temp/humidity sensor?"
		
		input "dewpoint", "number", required: false,
			title: "Dew Point Threshold (default 70)?"
	}

}
```

An interesting note about the `outdoor` input defined there is the "capability" string. SmartThings will use the specified capability to build a list of potential targets for this input. Specifically, every device I have installed that reports a humidity measurement capability will be available to pick when this app is installed and configured.

In a little while when it's time to discover the outdoor dew point measurement, I'm going to need humidity *and* temperature, so ideally I would pick a device that can report both. Unfortunately, you can't specify multiple capabilities in an input, so my options are either use a second input for the temperature sensor or bank on the fact that I don't have any humidity sensors that aren't also temperature sensors. For simplicity, I'm doing the latter.

After updating the preferences section of the app, I need to consider another piece of code that I had previously written, the `turnOffERV()` method:

```groovy
def turnOffERV() {
	switches.each {
    	log.debug "Turning off ${it}."
		it.off()
	}
}
```

This method worked really well when I was using a group of `switches`, but now I have discrete switches and this method needs to be updated so it won't break:

```groovy
def turnOffERV() {
	log.debug "Turning off ERV switches."
	erv.off()
	hilo.off()
}
```

Not a big change. I have two switches named `erv` and `hilo`, and I send each of them the `off()` command.

# Scheduling Periodic Ventilation

Now that I have split my switches up and set up some preferences, I turn my attention to the `updated` and `initialize` methods of my SmartApp:

```groovy
def updated() {
	log.debug "Updated with settings: ${settings}"

	unsubscribe()
	initialize()
}

def initialize() {
	subscribe(location, "routineExecuted", routineChanged)
}
```

This SmartApp originally had to only react to executed Routines and do something when one routine in particular was executed. Now I have something else to do, which I specified earlier: "Execute continuously, not in response to executing a routine or flipping a switch."

I make a couple of changes to these two methods:

```groovy
def updated() {
	log.debug "Updated with settings: ${settings}"

	unsubscribe()
	unschedule()
	initialize()
}

def initialize() {
	runEvery1Hour(periodicVentilation)
	subscribe(location, "routineExecuted", routineChanged)
}
```

In `updated()` I've added the command `unschedule()` which simply unschedules any jobs that this SmartApp has scheduled. The other change is the `runEvery1Hour` method call in the `initialize` method.

This `runEvery1Hour` method is a helper method that is part of the SmartThings platform. While you can craft entire `cron` expressions in a call to `schedule()`, using one of the helpers lets SmartThings randomly pick a time in the next hour. A job scheduled with `runEvery1Hour` might execute at any time, but it will execute at that same time every hour, whether it's at 23 minutes or 41 minutes past the hour.

Now I need to define what it's actually going to do every hour by writing the `periodicVentilation` method.

# The Periodic Ventilation Method

This method is going to execute every hour, but I need to understand what it's meant to do:

* Check outdoor dew point value.
* Turn on ERV (low-speed).
* Schedule a time n minutes in the future to turn the ERV off.

Seems simple enough. There's at least enough detail to get started.

The first hurdle is the toughest. How do I check the outdoor Dew Point? My sensor will give me Temperature (in Fahrenheit) and Relative Humidity, but not Dew Point. Turns out that's all you really need, and there's a simple back-of-the-envelope approximation calculation to get Dew Point from these two values:

$$ DP = T_F - \frac{9}{25} (100 - RH) $$

That's as good as this needs to be, so I'll implement this math and the necessary code to use it.

```groovy
def periodicVentilation() {

	// Check outdoor Dew Point, and if it is above threshold, don't turn on the ERV.
	if ( getCurrentDewPoint() < getDewPointThreshold() ) {
		// turn on the ERV's low-speed mode.
		erv.on()
		hilo.off()
		// Using the 'pminutes' input, schedule a time to turn off the ERV
		runIn(60 * findRunTime(), turnOffERV)
	} else {
		log.debug("Dew Point is too high for ventilation.")
		if (erv.currentSwitch == "on" ) { turnOffERV() }
		return false
	}
}

def getCurrentDewPoint() {
	def T = outdoor.currentValue("temperature")
	def RH = outdoor.currentValue("humidity")
	def DP = T - ( 9/25 * ( 100 - RH ) )
	
	log.debug("Current outdoor dew point is ${DP}")
	
	DP
}

def getDewPointThreshold() {
	(dewpoint != null && dewpoint !="") ? dewpoint : 70
}

//Validate runtime in minutes
private findRunTime() {
	if (pminutes > 60) {
		return 60
    } else if (pminutes < 1) {
    	return 1 
    } else {
    	return pminutes
    }
}
```

There's a lot going on there, but it should be reasonably straightforward. There are issues, and I need to address these before I publish this SmartApp to my platform. Even though I suspect this is a working prototype, it can and should be a little smarter.

Consider:

* If the outdoor sensor is unavailable (maybe its battery is dead, or the network can't reach it, etc.), what happens?
* If I'm running the ERV Timed Ventilation routine, should the Periodic Ventilation trump it or allow it to operate?
* Both the `periodicVentilation` and `routineChanged` methods schedule the execution of the `turnOffERV` method. Do these methods work together or against each other?

# SmartApp State

Let's consider the case of the outdoor sensor being unavailable. It's bound to happen. My outdoor sensor is actually the outdoor module of a [Netatmo Weather Station][] that's connected to my SmartThings instance. There are all sorts of reasons this connection might get broken or the module become unresponsive. For this reason, I want my SmartApp to cache the current outdoor conditions, and I want a backup plan.

Every SmartApp has available to it a private key-value store called the `state`. I can poke any values I want into this cache and retrieve them later. I'm going to use this feature to make the outdoor sensor measurements separate from the ventilation logic.

First, I need to `subscribe` to the outdoor sensor's temperature and humidity measurements. These subscriptions will call an event handler to update the `state` available to my app. I already added this input to my `preferences`, and named it "outdoor," so I'm good to go in that respect.

```groovy
def initialize() {
	//...
	subscribe(outdoor, "temperature", outdoorHandler)
	subscribe(outdoor, "humidity", outdoorHandler)
}

def outdoorHandler(evt) {
	//Calculate dew point
    def T = outdoor.currentValue("temperature")
    def RH = outdoor.currentValue("humidity")
    def DP = T - ( 9/25 * (100-RH) )
    log.debug("Current outdoor Dew Point is $DP")
    
    state.dewpoint = DP
    state.outtemp = T
    state.outrh = RH
    state.lastForecast = now() / 1000
    state.forecastSource = "Sensor"
}
```

Now, any time the outdoor module reports either the temperature or humidity, my app will calculate a dew point and store the temperature, humidity, and dew point values in the application's `state`, along with a timestamp as "lastForecast" in Epoch time and an identifier that the source of this data ("forecastSource") is the "Sensor".

Now I can modify the `periodicVentilation` method to use this stored state data by modifying the `getCurrentDewPoint` method:

```groovy
def getCurrentDewPoint() {
	state.dewpoint
}
```

It's a little funny that my Netatmo already knows the dew point; I can open my Netatmo app and see it. It probably does the same math. But I can't ask it for that value, so I take what I can get and do the math myself.

# Next Time

In Part 3 of this series, I'll address the need for a backup plan.

<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

[Forecast.io]: https://developer.forecast.io
[Netatmo Weather Station]: /the_tools/netatmo
[Part 1]: {% link _the_erv/Automating-ERV.md %}
[Z-Wave functionality]: {% link _the_erv/ERV-switching.md %}
[ERV]: /use_cases/the_erv.html
[SmartThings]: /the_tools/smartthings.html
[Dew Point]: https://en.wikipedia.org/wiki/Dew_point
[Github repository]: https://github.com/tcjennings/SmartThingsPrivate/blob/53c9cc6266d57373e7454d20008e293353004d05/smartapps/tcjennings/erv-timed-operation.src/erv-timed-operation.groovy
[complete SmartApp]: https://github.com/tcjennings/SmartThingsPrivate/blob/3d4e211752f5350a0ec0f15ad5e04a52e4cc2c50/smartapps/tcjennings/erv-timed-operation.src/erv-timed-operation.groovy
