node-weatherboard
=================

*This project is a placeholder for my notes until I actually move to my farm, buy the equipment from sparkfun, and write the code.*

Use the sparkfun usb weather board in node.

## Desired API:

```javascript

var WeatherBoard = require('weatherboard');

weather = new WeatherBoard("/dev/tty-usbserial1", {baudrate: 9600, units: "English", samplingRate: 2});

// Data events are useful for dashboard-like displays that show realtime data. WeatherBoard parses CSV data
// and provides a javascript object
weather.on("data", function(data){
  console.log("The current wind direction is " + data.windDirection + " deg.");
});

// WeatherBoard.every() will average values for the given sampling period. Useful for sending values
// to a database.
weather.every("1m", function(data){
  console.log("The average temperature in the last 1 minute has been: " + data.temperature + " deg F.");
});

// Will fire whenever a given sampling period defined by .every() came up empty
weather.on("nodata", function(duration, lastReadingAt){
  var minutes = Math.round(duration/60);
  console.log("No sensor readings have been transmitted for " + minutes + " minutes.");
  console.log("Last reading was made at " + lastReadingAt);
});

// What would be the error conditions exactly?
weather.on("error", function(err){
  console.log("Error processing sensor stream", err);
});
```

## Implementation

  * weather.every would be syntactic sugar for creating a new Transform-type Stream that averages all values sampled for a given duration. This stream interface could be exposed to end-users, or the every() method could even be removed in favor of streams.
  * WeatherBoard will be designed with the purpose of enabling developers to create reliable, long-running daemons for saving information to a database. Dashboard applications should then be written as a separate application that pulls information  from these databases. For example, sending hourly averages to postgres and realtime data to redis. Then another application could use redis pubsub and websockets to show a realtime dashboard + historical data from postgres.
  * WeatherBoard streams might provide more than just averages. For example, min and max values for a sampling period, and the number of samples.
