---
title: Weather
name: weather
type: app
version: 0.1.0
---

# Weather

A three-action weather app, backed by [wttr.in](https://wttr.in) for the
weather data and [Nominatim](https://nominatim.openstreetmap.org/) (OpenStreetMap)
for resolving city names. No API key, no signup, no server to run.
Works the moment it is installed.

## Actions

- `/act.now <city>` — current conditions, one line
- `/act.forecast <city>` — 3-day forecast with 4 timesteps per day, compact
  agent-friendly format
- `/act.search <query>` — resolve a free-form location query (city,
  country, landmark, airport code, ZIP, GPS) to canonical names you can
  pass to `now` / `forecast`. Use this first when the user's location
  is ambiguous (e.g. *"Springfield"*, *"Cambridge"*) or transliterated.

Multi-word cities: quote them — `/act.now "New York"` — or use `+` in
place of spaces. Flag form (`--city New+York`) also works.

```act.now
GET https://wttr.in/{city}?format=%l:+%C+%t+%w&m -H "User-Agent: curl/8"
  city: string (required) "City name"
```

```act.forecast
GET https://wttr.in/{city}?format=j1&m -H "User-Agent: curl/8"
  city: string (required) "City name"
```

```act.forecast.response
{city} = {Response.body.nearest_area[0].areaName[0].value}
{country} = {Response.body.nearest_area[0].country[0].value}
**{city}, {country}** — 3-day forecast (°C, km/h, mm rain)

for: day in Response.body.weather
### {day.date} ({day.mintempC}–{day.maxtempC}°C, avg {day.avgtempC}°C)
- 06: {day.hourly[2].tempC}°C, {day.hourly[2].weatherDesc[0].value}, wind {day.hourly[2].winddir16Point} {day.hourly[2].windspeedKmph}km/h, rain {day.hourly[2].chanceofrain}% ({day.hourly[2].precipMM}mm)
- 12: {day.hourly[4].tempC}°C, {day.hourly[4].weatherDesc[0].value}, wind {day.hourly[4].winddir16Point} {day.hourly[4].windspeedKmph}km/h, rain {day.hourly[4].chanceofrain}% ({day.hourly[4].precipMM}mm)
- 18: {day.hourly[6].tempC}°C, {day.hourly[6].weatherDesc[0].value}, wind {day.hourly[6].winddir16Point} {day.hourly[6].windspeedKmph}km/h, rain {day.hourly[6].chanceofrain}% ({day.hourly[6].precipMM}mm)
- 21: {day.hourly[7].tempC}°C, {day.hourly[7].weatherDesc[0].value}, wind {day.hourly[7].winddir16Point} {day.hourly[7].windspeedKmph}km/h, rain {day.hourly[7].chanceofrain}% ({day.hourly[7].precipMM}mm)
- sun {day.astronomy[0].sunrise} → {day.astronomy[0].sunset}
end:
```

```act.search
GET https://nominatim.openstreetmap.org/search?format=json&limit=5 -H "User-Agent: string-cookbook-weather/0.1"
  q: string (required) "Free-form location query"
```

```act.search.response
{top} = {Response.body[0].display_name}
{lat} = {Response.body[0].lat}
{lon} = {Response.body[0].lon}
Top match: {top}
Coordinates: {lat}, {lon}

Other matches:
- {Response.body[1].display_name}
- {Response.body[2].display_name}
- {Response.body[3].display_name}
- {Response.body[4].display_name}

(Pass the top match to /act.now "<top>", or coordinates as /act.now {lat},{lon})
```
