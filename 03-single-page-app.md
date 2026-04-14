# Cookbook: Single-Page App

A complete walkthrough of building a single-file SFMD application —
a weather dashboard where everything lives in one document. The AI
never leaves the page; response templates are how the view changes.

**Scenario:** Build a weather app that searches cities, shows current
conditions and forecasts, and creates weather alerts — all in one file.

> See [Actions](https://github.com/string-os/string/blob/main/docs/runtime/05-actions.md) for action definitions.
> See [State](https://github.com/string-os/string/blob/main/docs/runtime/06-state.md) for variables and `/set`.

---

## The document IS the app

A single-page app is one SFMD file. It contains:
- Content the AI reads (action hints, instructions)
- Action definitions (hidden from AI, parsed by String)
- Response templates (format output, store variables)

No navigation, no directory structure, no nav menus.
All interaction happens through `/act`.

---

## 1. Create the app

One file: `~/apps/weather.md`.

**AI →**

```
<𝒞=string:~/apps/weather.md>
---
title: Weather Dashboard
---

# Weather Dashboard

Search for a city, check conditions, view forecasts, and set alerts.

**Commands:**
`/act.search_city --name "{City Name}"`
`/act.current --city "{city}"`
`/act.forecast --city "{city}" --days "7"`
`/act.create_alert --city "{city}" --condition "{condition}"`

## Weather
Search for a city to get started.

```act.search_city
GET https://api.weather.com/v2/search
  name: string (required) "City name to search"
```

```act.search_city.response
{city} = {Response.body.city}
{country} = {Response.body.country}
{lat} = {Response.body.lat}
{lon} = {Response.body.lon}
## Found: {city}, {country}
- Coordinates: {lat}, {lon}
```

```act.current
GET https://api.weather.com/v2/current
  city: string (required) "City name"
  unit: string (optional) "celsius|fahrenheit"
```

```act.current.response
{temp} = {Response.body.temperature}
{condition} = {Response.body.condition}
{humidity} = {Response.body.humidity}
## {Response.body.city}
- **Temperature:** {temp}°{Response.body.unit}
- **Condition:** {condition}
- **Humidity:** {humidity}%
- **Wind:** {Response.body.wind_speed} m/s ({Response.body.wind_dir})
```

```act.forecast
GET https://api.weather.com/v2/forecast
  city: string (required) "City name"
  days: number (optional) "Number of days (default: 7)"
  unit: string (optional) "celsius|fahrenheit"
```

```act.forecast.response
## {Response.body.city} — {Response.body.days}-Day Forecast

{Response.body.forecast}

**Average:** {Response.body.avg_temp}° | **Range:** {Response.body.min_temp}°–{Response.body.max_temp}°
```

```act.create_alert
POST https://api.weather.com/v2/alerts
  city: string (required) "City name"
  condition: string (required) "Condition to alert on (rain, snow, heat, wind)"
```

```act.create_alert.response
{alert_id} = {Response.body.id}
✓ Alert created (ID: {alert_id})
- City: {Response.body.city}
- Condition: {Response.body.condition}
- Status: active
```
</𝒞>
```

**String →**

```
<𝒞=string:~/apps/weather.md>
✓ created ~/apps/weather.md (68 lines)
</𝒞>
```

Everything in one file — content, four actions, four response templates.

### What the AI sees

When the AI opens this file, String hides the `act.*` code blocks
and shows only the content:

```
# Weather Dashboard

Search for a city, check conditions, view forecasts, and set alerts.

**Commands:**
`/act.search_city --name "{City Name}"`
`/act.current --city "{city}"`
`/act.forecast --city "{city}" --days "7"`
`/act.create_alert --city "{city}" --condition "{condition}"`

## Weather
Search for a city to get started.
```

The action hints tell the AI what it can do. The action specs tell
String how to execute them. Clean separation.

---

## 2. Open the app

**AI →**

```
<𝒞=string:app:weather>
/open ~/apps/weather.md
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:00+09:00">
# Weather Dashboard

Search for a city, check conditions, view forecasts, and set alerts.

**Commands:**
`/act.search_city --name "{City Name}"`
`/act.current --city "{city}"`
`/act.forecast --city "{city}" --days "7"`
`/act.create_alert --city "{city}" --condition "{condition}"`

## Weather
Search for a city to get started.
</𝒞>
```

The AI sees the content and knows what actions are available.

---

## 3. Search a city

**AI →**

```
<𝒞=string:app:weather>
/act.search_city --name "Seoul"
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:00+09:00">
## Found: Seoul, South Korea
- Coordinates: 37.5665, 126.9780
</𝒞>
```

Variables stored: `{city}` = "Seoul", `{country}` = "South Korea",
`{lat}` = "37.5665", `{lon}` = "126.9780".

The response template formatted the output and stored four variables
for later use.

---

## 4. Current weather

The AI chains into the next action using `{city}` from the search.

**AI →**

```
<𝒞=string:app:weather>
/act.current --city {city}
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:01+09:00">
## Seoul
- **Temperature:** 18°celsius
- **Condition:** Partly Cloudy
- **Humidity:** 52%
- **Wind:** 3.2 m/s (NW)
</𝒞>
```

String substituted `{city}` → "Seoul" at runtime. The response
template stored `{temp}`, `{condition}`, and `{humidity}`.

---

## 5. 7-day forecast

Still on the same page — no navigation needed.

**AI →**

```
<𝒞=string:app:weather>
/act.forecast --city {city} --days "7"
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:01+09:00">
## Seoul — 7-Day Forecast

| Day | High | Low | Condition |
|-----|------|-----|-----------|
| Wed | 18° | 10° | Partly Cloudy |
| Thu | 20° | 12° | Clear |
| Fri | 22° | 13° | Clear |
| Sat | 19° | 11° | Rain |
| Sun | 16° | 9° | Cloudy |
| Mon | 17° | 10° | Partly Cloudy |
| Tue | 21° | 12° | Clear |

**Average:** 19° | **Range:** 9°–22°
</𝒞>
```

In a multi-page app, the forecast would be a separate page reached
with `/open`. Here, it's just another `/act` on the same page.
The response template IS the view.

---

## 6. Create an alert

**AI →**

```
<𝒞=string:app:weather>
/act.create_alert --city {city} --condition "rain"
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:02+09:00">
✓ Alert created (ID: alert_7291)
- City: Seoul
- Condition: rain
- Status: active
</𝒞>
```

Four actions, all on one page, no navigation. The response templates
handle everything — formatting output, storing variables, confirming
mutations.

---

## 7. State flow

Variables chain between actions. Here's the full flow:

```
/act.search_city --name "Seoul"
  → stores {city}, {country}, {lat}, {lon}

/act.current --city {city}
  → uses {city} from search
  → stores {temp}, {condition}, {humidity}

/act.forecast --city {city}
  → uses {city} from search

/act.create_alert --city {city} --condition "rain"
  → uses {city} from search
  → stores {alert_id}
```

Each action reads variables set by previous actions. The AI doesn't
manage state manually — it just uses `{city}` and String substitutes
the stored value.

### Check state with /info

**AI →**

```
<𝒞=string:app:weather>
/info
</𝒞>
```

**String →**

```
<𝒞=string:app:weather>
topic:   app:weather
type:     app
uri:      ~/apps/weather.md
history:  1
variables:
  {city} = "Seoul"
  {country} = "South Korea"
  {lat} = "37.5665"
  {lon} = "126.9780"
  {temp} = "18"
  {condition} = "Partly Cloudy"
  {humidity} = "52"
  {alert_id} = "alert_7291"
actions:  search_city, current, forecast, create_alert
</𝒞>
```

One page, one topic, all state in one place.

---

## 8. App configs — same app, different topics

The same weather file can serve different regions. Each `app:weather:config`
topic gets its own state.

### Korea config

**AI →**

```
<𝒞=string:app:weather:korea>
/open ~/apps/weather.md
</𝒞>
```

```
<𝒞=string:app:weather:korea>
/set
{unit} = "celsius"
{default_city} = "Seoul"
</𝒞>
```

### Japan config

**AI →**

```
<𝒞=string:app:weather:japan>
/open ~/apps/weather.md
</𝒞>
```

```
<𝒞=string:app:weather:japan>
/set
{unit} = "celsius"
{default_city} = "Tokyo"
</𝒞>
```

### Use them side by side

**AI →**

```
<𝒞=string:app:weather:korea>
/act.current --city {default_city}
</𝒞>

<𝒞=string:app:weather:japan>
/act.current --city {default_city}
</𝒞>
```

**String →**

```
<𝒞=string:app:weather:korea time="2026-03-19T11:10+09:00">
## Seoul
- **Temperature:** 18°celsius
- **Condition:** Partly Cloudy
- **Humidity:** 52%
- **Wind:** 3.2 m/s (NW)
</𝒞>

<𝒞=string:app:weather:japan time="2026-03-19T11:10+09:00">
## Tokyo
- **Temperature:** 22°celsius
- **Condition:** Clear
- **Humidity:** 41%
- **Wind:** 2.1 m/s (S)
</𝒞>
```

Same file, different topics, different state. The `app:weather:korea`
and `app:weather:japan` topics are fully independent.

> See [Topics](https://github.com/string-os/string/blob/main/docs/runtime/04-topics.md) for the `app:name:config` model.

---

## When to use a single-page app

| Use case | Why single-page |
|----------|-----------------|
| **Tool wrappers** | One API, a few actions, no sections needed |
| **Dashboards** | All data shown inline via response templates |
| **Quick search** | Search + results on the same page |
| **Simple workflows** | Linear action chains, no branching |

When sections grow large or actions become many, consider a
[multi-page app](./04-multi-page-app.md) instead.

---

## Summary

| Pattern | How |
|---------|-----|
| **One file** | Everything in `~/apps/weather.md` |
| **Actions** | `` ```act.name `` code blocks, hidden from AI |
| **Response templates** | `` ```act.name.response `` — format output + store variables |
| **State flow** | `{var}` stored by responses, passed between actions |
| **View changes** | Response templates, not page navigation |
| **Config** | `app:name:config` — same file, different topics |

The single-page flow: **open → /act → read response → /act again.**
All `/act`, no `/open`. The document IS the app.
