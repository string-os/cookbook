---
title: Weather
name: weather
type: app
version: 0.3.0
default: now
requires:
  - CITY
description: |
  Three-action weather app — current, 3-day forecast, free-form
  location search. Backed by wttr.in and OpenStreetMap Nominatim.
  No API key. Per-config $CITY for one-touch dashboards.
---

# Weather

Backed by [wttr.in](https://wttr.in) for weather and
[Nominatim](https://nominatim.openstreetmap.org/) (OpenStreetMap) for
resolving free-form locations. No API key, no signup.

## Per-config dashboards

Each config gets its own `$CITY`. Set once, open instantly:

```
string app:weather:seoul '/set $CITY = "seoul"'
string '/open app:weather:seoul'                  # → Seoul weather, no args
string '/open app:weather:tokyo'                  # ✗ asks you to set $CITY
```

## Actions

- `/act.now [city]` — current conditions (uses `$CITY` if no arg)
- `/act.forecast [city]` — 3-day forecast
- `/act.search <query>` — resolve ambiguous queries to 5 candidates (registers `@city-N`)

Multi-word cities: quote them — `/act.now "New York"` — or use `+` for spaces.

```act.now
GET https://wttr.in/{city}?format=%l:+%C+%t+%w&m -H "User-Agent: curl/8"
  city: string "City name or lat,lon" = "$CITY"
```

```act.now.response
{Response.body}

next: /act.forecast {city} for 3-day · /act.search "..." for ambiguous names
```

```act.forecast
GET https://wttr.in/{city}?format=j1&m -H "User-Agent: curl/8"
  city: string "City name or lat,lon" = "$CITY"
```

```act.forecast.response
{area} = {Response.body.nearest_area[0].areaName[0].value}
{country} = {Response.body.nearest_area[0].country[0].value}
**{area}, {country}** — 3-day forecast (°C, km/h, mm rain)

for: day in Response.body.weather
### {day.date} ({day.mintempC}–{day.maxtempC}°C, avg {day.avgtempC}°C)
- 06: {day.hourly[2].tempC}°C, {day.hourly[2].weatherDesc[0].value}, wind {day.hourly[2].winddir16Point} {day.hourly[2].windspeedKmph}km/h, rain {day.hourly[2].chanceofrain}% ({day.hourly[2].precipMM}mm)
- 12: {day.hourly[4].tempC}°C, {day.hourly[4].weatherDesc[0].value}, wind {day.hourly[4].winddir16Point} {day.hourly[4].windspeedKmph}km/h, rain {day.hourly[4].chanceofrain}% ({day.hourly[4].precipMM}mm)
- 18: {day.hourly[6].tempC}°C, {day.hourly[6].weatherDesc[0].value}, wind {day.hourly[6].winddir16Point} {day.hourly[6].windspeedKmph}km/h, rain {day.hourly[6].chanceofrain}% ({day.hourly[6].precipMM}mm)
- 21: {day.hourly[7].tempC}°C, {day.hourly[7].weatherDesc[0].value}, wind {day.hourly[7].winddir16Point} {day.hourly[7].windspeedKmph}km/h, rain {day.hourly[7].chanceofrain}% ({day.hourly[7].precipMM}mm)
- sun {day.astronomy[0].sunrise} → {day.astronomy[0].sunset}
end:

next: /act.now {area} for current · /act.forecast <other-city>
```

```act.search
GET https://nominatim.openstreetmap.org/search?format=json&limit=5&q={q} -H "User-Agent: string-cookbook-weather/0.2"
  q: string (required) "Free-form location query"
```

```act.search.response
Search: "{q}"

for: r in Response.body
{@city} = {r.lat},{r.lon}
- {@city}: {r.display_name}
end:

next: /act.now @city-N · /act.forecast @city-N
```
