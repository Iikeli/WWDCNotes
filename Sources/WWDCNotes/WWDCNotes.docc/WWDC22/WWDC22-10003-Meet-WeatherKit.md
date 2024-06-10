# Meet WeatherKit

WeatherKit offers valuable weather data for your apps and services to help people stay up to date on the latest conditions.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10003", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- [new framework][wk]
- powered by Apple Weather Service
- uses high-resolution weather models
- uses machine learning and prediction algorithms

## Available datasets

Data available:

- Current weather
- Minute forecast
- Hourly
- forecast
- Daily forecast
- Weather alerts
- Historical
- weather
- UV index
- Humidity
- Pressure
- Moonset
- Precipitation chance
- Cloud cover
- Visibility
- Condition
- temperature
- Wind gust
- Sunrise
- Pressure trend /
- Wind speed
- Moonrise
- Weather severity
- Precipitation
- intensity
- Precipitation amount
- High temperature
- Wind
- direction
- Low temperature
- Dew point
- Apparent
- temperature
- Moon phase
- Sunset
- Snowfall amount

Data sets:

- Current weather
  - "now" conditions at the requested location
  - represents a single point in time and includes conditions like UV index, temperature, and wind

- Minute forecast
  - minute-by-minute precipitation conditions for the next hour

- Hourly forecast
  - collection of forecasts starting on the current hour
  - provides data for up to 240 hours
  - each hour includes conditions like humidity, visibility, pressure, and dew point

- Daily forecast
  - forecast collection of 10 days
  - each day provides information about the entire day, like the high and low temperature, sunrise, and sunset

- Weather alerts
  - severe weather warnings issued for the requested location
  - important information to keep your users safe, informed, and prepared

- Historical weather
  - saved weather forecasts from the past
  - you can access historical data by specifying a start and end date to the hourly and daily requests

## Requesting weather

Two ways:

- [WeatherKit][wk]
- [WeatherKit REST API][api]

### WeatherKit

```swift
import WeatherKit
import CoreLocation

let weatherService = WeatherService()
let location = CLLocation(latitude: 43, longitude: -76)
let weather = try! await weatherService.weather(for: location)
let temperature = weather.currentWeather.temperature
let uvIndex = weather.currentWeather.uvIndex
```

### WeatherKit REST API

```js
/* Request a token */
const tokenResponse = await fetch('https://example.com/token');
const token = await tokenResponse.text();

/* Get my weather object */
const url = "https://weatherkit.apple.com/1/weather/en-US/41.029/-74.642?dataSets=weatherAlerts&country=US"

const weatherResponse = await fetch(url, {
  headers: {
    "Authorization": token
  }
});
const weather = await weatherResponse.json();

/* Check for active weather alerts */
const alerts = weather.weatherAlerts;
const detailsUrl = weather.weatherAlerts.detailsUrl;
```
## Publishing requirements

1. attribution: you must display a link that is retrieved from the Attribution APIs
2. attribution logo: you must display the Apple Weather logo, given both by WeatherKit and the rest API
3. if you show weather alters: you must link to a Weather alert page included in the response

[wk]: https://developer.apple.com/documentation/WeatherKit
[api]: https://developer.apple.com/documentation/weatherkitrestapi