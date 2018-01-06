---
title: Automating the ERV Part 3
layout: page
tags: erv smartthings
---

In [Part 1][] I leveraged the [Z-Wave functionality][] I'd added to my [ERV][] to create some basic automation for its operation. I implemented a [SmartThings][] routine to begin high-speed ventilation on demand and, not liking the open-endedness of that, wrote a Smart App to shut the thing down again after some amount of time.

In [Part 2][] I built on this foundation and added some functionality that would run my ventilator for *n* minutes of every hour depending on outdoor conditions as determined by a temperature and humidity sensor.

Now it's time to go one step further and implement a backup plan.

# The Backup Plan

It might be the case that the values stored in `state` become stale because the outdoor module stops working, in which case subscription events stop occurring and the handler method that updates the `state` no longer executes. In this case I want to fall back to a Web weather API from [Forecast.io][]. Since I'm storing a timestamp along with the other values in `state`, I can figure out how fresh that reading is. Let's say I want to fall back to the web if the `state` values are more than 45 minutes old.

Forecast.io has a developer program that allows 1000 API hits a day, which is more than enough for private residential use. I'll use this more down the line when I start putting together logic for my [irrigation][] use case, but for today I'll use Forecast.io's current conditions data.

I will update the `getCurrentDewPoint` method in my SmartApp to check the application state for staleness.

```groovy
def getCurrentDewPoint() {
	def lastForecast = state.lastForecast
	def staleThreshold = ( now() / 1000 ) - ( 60 * 45 )
	def DP
	if ( lastForecast < staleThreshold ) {
		DP = getWebForecast()
	} else {
		DP = state.dewpoint
	}
	
	DP
}
```

Now the method returns the dewpoint value from `state` only if that value is reasonably fresh. Otherwise it calls the `getWebForecast()` method, which I now need to write and make sure it returns a dew point value.

An interesting note about the above code -- in determining the `staleThreshold` I use the `now()` method, which returns the current time as Epoch time (the number of seconds since midnight January 1, 1970) with a millisecond resolution. The subsequent division by 1000 removes the millisecond precision and gives me the time in seconds. Then I subtract 45 minutes (in seconds) from the current time to get my threshold for staleness, i.e., the earliest timestamp which I will consider "fresh".

To use the Forecast.io API my app is going to have to know my private API key and the latitude and longitude of my location. The way you introduce private settings into a SmartApp is in its `definition` section, which is kind of its preamble. Recall I originally created this SmartApp "From Form," and that form set up the definition for me.

I will add three private settings for this SmartApp:

```groovy
definition(
	//...
) {
	appSetting "apikey"
	appSetting "latitude"
	appSetting "longitude"
}
```

Note the unusual specification of these settings; they are specified in a closure (in `{...}`) associated with the `definition()` method.

With those defined, I first have to populate the values, which is done in the SmartThings IDE, in the "Settings" section of the SmartApp; then I can access them through the `appSettings` Groovy map, as you'll see in the URL constructor in this `getWebForecast()` method definition:

```groovy
def getWebForecast() {
	def forecastURL = "https://api.forecast.io/forecast/${appSettings.apikey}/${appSettings.latitude},${appSettings.longitude}"
	def responseTime
	def responseDewPoint
	def responseTemp
	def responseRH
	
	log.debug "Checking Forecast.io weather"

	httpGet(forecastURL) { response ->
		if (response.data) {
			responseTime = response.data.currently.time.intValue()
			responseDewPoint = response.data.currently.dewPoint.floatValue()
			responseTemp = response.data.currently.temperature.floatValue()
			responseRH = response.data.currently.humidity.floatValue()
			
			state.lastForecast = responseTime
			state.forecastSource = "Web"
			
			state.dewpoint = responseDewPoint
			state.outtemp = responseTemp
			state.outrh = responseRH
		} else {
			responseDewPoint = 100
			
			state.forecastSource = "Unavailable"
			state.dewpoint = responseDewPoint
			
			log.debug "HttpGet Response data unsuccessful"
		}
	}
	responseDewPoint
}
```

That's a lot of stuff, but ought to be easy to follow. The SmartApp [documentation for using Web Services](http://docs.smartthings.com/en/latest/smartapp-developers-guide/calling-web-services-in-smartapps.html) is comprehensive on this topic. The response from the Forecast.io API call is a JSON document, which is parsed automatically for us, so in the closure associated with the `httpGet` call we can read directly from `response.data` and pull out any values we want. The JSON returned by Forecast.io is [very large](https://developer.forecast.io/docs/v2#forecast_call) and contains multiple forecasts. Here I want only the values that represent conditions "currently" in effect. Then I update the application's `state` with the Forecast.io values and return the dew point measurement.

In case the HTTP call should fail, notice that a value of 100 will be returned for the dew point. This should effectively disable periodic ventilation; when all else fails we'll err on the side of not running the ventilator. I might want to add a push notification here, too, so I can be made aware that this thing is not working.

One last thing: I want to make this `getWebForecast` method part of the `initialize` part of the SmartApp, so that the application state will be primed with some data when it is first installed and configured.

```groovy
def initialize() {
	getWebForecast()
	runEvery1Hour(periodicVentilation)
	subscribe(location, "routineExecuted", routineChanged)
	subscribe(outdoor, "temperature", outdoorHandler)
	subscribe(outdoor, "humidity", outdoorHandler)
}
```

# Next Time

In Part 4 of this series, I'll address the coexistence of the periodic and on-demand ventilation rules.

Here is the [complete SmartApp][] so far.

[Part 1]: {% link collections/_the_erv/Automating-ERV.md %}
[Part 2]: {% link collections/_the_erv/Automating-ERV-2.md %}
[Z-Wave functionality]: {% link collections/_the_erv/ERV-switching.md %}
[ERV]: /use_cases/the_erv.html
[SmartThings]: /the_tools/smartthings.html
[Forecast.io]: https://developer.forecast.io
[irrigation]: /use_cases/the_sprinklers.html
[complete SmartApp]: https://github.com/tcjennings/SmartThingsPrivate/blob/3d4e211752f5350a0ec0f15ad5e04a52e4cc2c50/smartapps/tcjennings/erv-timed-operation.src/erv-timed-operation.groovy
