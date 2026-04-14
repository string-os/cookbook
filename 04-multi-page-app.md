# Cookbook: Multi-Page App

A complete walkthrough of building a multi-file SFMD application —
a weather dashboard split across pages with shared navigation.
`/open` to move between pages, `/act` to do things on each page.

**Scenario:** Build the same weather app as the
[single-page guide](./03-single-page-app.md), but structured as a
directory with separate pages for search, forecast, alerts, and settings.

> See [Actions](https://github.com/string-os/string/blob/main/docs/runtime/05-actions.md) for action definitions.
> See [State](https://github.com/string-os/string/blob/main/docs/runtime/06-state.md) for variables and `/set`.
> See [Navigation](https://github.com/string-os/string/blob/main/docs/runtime/07-navigation.md) for menus and shortcuts.

---

## Topic directory structure

```
~/apps/weather/
├── index.md        # Home — search + current weather
├── forecast.md     # 7-day forecast
├── alerts.md       # Alert management
├── settings.md     # Configuration display
└── nav/
    └── main.md     # @home, @forecast, @alerts, @settings
```

Each page has focused responsibility. Navigation ties them together.

---

## 1. Create the navigation menu

Start with the nav file — every page references it.

**AI →**

```
<𝒞=string:~/apps/weather/nav/main.md>
[@home Home](../index.md)
[@forecast Forecast](../forecast.md)
[@alerts Alerts](../alerts.md)
[@settings Settings](../settings.md)
</𝒞>
```

**String →**

```
<𝒞=string:~/apps/weather/nav/main.md>
✓ created ~/apps/weather/nav/main.md (4 lines)
</𝒞>
```

Four shortcuts. Paths are relative to the nav file's location.

---

## 2. Create the home page

The main page: search and current weather actions.

**AI →**

```
<𝒞=string:~/apps/weather/index.md>
---
title: Weather Dashboard
---

[!nav:main](./nav/main.md)

# Weather Dashboard

Welcome to the Weather Dashboard.

**Search for a city:**
`/act.search_city --name "{City Name}"`

**Get current conditions:**
`/act.current --city "{city}"`

## Current Weather
Search for a city to see weather data.

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
- [View Forecast][@main.forecast] | [Set Alert][@main.alerts]
```

```act.current
GET https://api.weather.com/v2/current
  city: string (required) "City name"
  unit: string (optional) "celsius|fahrenheit"
```

```act.current.response
{temp} = {Response.body.temperature}
{condition} = {Response.body.condition}
## {Response.body.city}
- **Temperature:** {temp}°{Response.body.unit}
- **Condition:** {condition}
- **Humidity:** {Response.body.humidity}%
- **Wind:** {Response.body.wind_speed} m/s ({Response.body.wind_dir})
```
</𝒞>
```

**String →**

```
<𝒞=string:~/apps/weather/index.md>
✓ created ~/apps/weather/index.md (48 lines)
</𝒞>
```

The response template for `search_city` includes navigation shortcuts —
`[@main.forecast]` and `[@main.alerts]` — so the AI knows where to go
next after finding a city.

---

## 3. Create the forecast page

A separate page with its own action for extended forecasts.

**AI →**

```
<𝒞=string:~/apps/weather/forecast.md>
---
title: 7-Day Forecast
---

[!nav:main](./nav/main.md)

# 7-Day Forecast

`/act.forecast --city "{city}" --days "7"`

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
</𝒞>
```

**String →**

```
<𝒞=string:~/apps/weather/forecast.md>
✓ created ~/apps/weather/forecast.md (21 lines)
</𝒞>
```

---

## 4. Create the alerts page

Alert management — create and list alerts, both on one page.

**AI →**

```
<𝒞=string:~/apps/weather/alerts.md>
---
title: Weather Alerts
---

[!nav:main](./nav/main.md)

# Weather Alerts

**Create an alert:**
`/act.create_alert --city "{city}" --condition "{condition}"`

**View active alerts:**
`/act.list_alerts --city "{city}"`

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

```act.list_alerts
GET https://api.weather.com/v2/alerts
  city: string (required) "City name"
```

```act.list_alerts.response
## Active Alerts — {Response.body.city}

{Response.body.alerts}

{Response.body.count} alert(s) active.
```
</𝒞>
```

**String →**

```
<𝒞=string:~/apps/weather/alerts.md>
✓ created ~/apps/weather/alerts.md (32 lines)
</𝒞>
```

---

## 5. Create the settings page

Displays configuration and tells the AI how to update it.

**AI →**

```
<𝒞=string:~/apps/weather/settings.md>
---
title: Settings
---

[!nav:main](./nav/main.md)

# Settings

Current configuration for this session:
- Unit: `{unit}`
- Default city: `{default_city}`
- Region: `{region}`

**Update settings:**
`/set` with the variables below.

Available variables:
- `{unit}` — `celsius` or `fahrenheit`
- `{default_city}` — city name for auto-load
- `{region}` — region code (e.g., KR, JP, US)
</𝒞>
```

**String →**

```
<𝒞=string:~/apps/weather/settings.md>
✓ created ~/apps/weather/settings.md (20 lines)
</𝒞>
```

No actions on this page — just state display. The AI uses `/set` to
update variables, then `/refresh` to see the changes.

---

## 6. Open the app and search

### Open the app

**AI →**

```
<𝒞=string:app:weather>
/open ~/apps/weather/index.md
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:00+09:00">
# Weather Dashboard

Welcome to the Weather Dashboard.

**Search for a city:**
`/act.search_city --name "{City Name}"`

**Get current conditions:**
`/act.current --city "{city}"`

## Current Weather
Search for a city to see weather data.
</𝒞>
```

### Search for a city

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
- [View Forecast][@main.forecast] | [Set Alert][@main.alerts]
</𝒞>
```

Variables stored: `{city}` = "Seoul", `{country}` = "South Korea".
The response includes navigation links to the forecast and alerts pages.

### Get current weather

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

So far this looks like a single-page app. The difference comes next.

---

## 7. Navigate to the forecast page

In the single-page app, the forecast is just another `/act`. Here,
it's a separate page the AI navigates to.

**AI →**

```
<𝒞=string:app:weather>
/open @main.forecast
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:02+09:00">
# 7-Day Forecast

`/act.forecast --city "{city}" --days "7"`
</𝒞>
```

The AI is now on `forecast.md`. The page has its own action and its
own content. But `{city}` is still "Seoul" — **state persists across
page navigations within the topic**.

### Run the forecast action

**AI →**

```
<𝒞=string:app:weather>
/act.forecast --city {city} --days "7"
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:02+09:00">
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

`{city}` carried over from the home page — variables persist within
the topic session.

---

## 8. Navigate to alerts and create one

**AI →**

```
<𝒞=string:app:weather>
/open @main.alerts
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:03+09:00">
# Weather Alerts

**Create an alert:**
`/act.create_alert --city "{city}" --condition "{condition}"`

**View active alerts:**
`/act.list_alerts --city "{city}"`
</𝒞>
```

### Create a rain alert

**AI →**

```
<𝒞=string:app:weather>
/act.create_alert --city {city} --condition "rain"
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:03+09:00">
✓ Alert created (ID: alert_7291)
- City: Seoul
- Condition: rain
- Status: active
</𝒞>
```

### List alerts

**AI →**

```
<𝒞=string:app:weather>
/act.list_alerts --city {city}
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:04+09:00">
## Active Alerts — Seoul

| ID | Condition | Status |
|----|-----------|--------|
| alert_7291 | rain | active |

1 alert(s) active.
</𝒞>
```

---

## 9. Navigate back to home

**AI →**

```
<𝒞=string:app:weather>
/back
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:04+09:00">
# 7-Day Forecast

`/act.forecast --city "{city}" --days "7"`
</𝒞>
```

**AI →**

```
<𝒞=string:app:weather>
/back
</𝒞>
```

**String →**

```
<𝒞=string:app:weather time="2026-03-19T11:04+09:00">
# Weather Dashboard

Welcome to the Weather Dashboard.
...
</𝒞>
```

`/back` walks through page history: alerts → forecast → home.
Or the AI can jump directly with `/open @main.home`.

### Check navigation

**AI →**

```
<𝒞=string:app:weather>
/nav
</𝒞>
```

**String →**

```
<𝒞=string:app:weather>
main:
  @main.home       Home           ← current
  @main.forecast   Forecast
  @main.alerts     Alerts
  @main.settings   Settings
</𝒞>
```

---

## 10. App configs — same app, different topics

Just like the single-page version, the multi-page app supports
per-config topics.

### Korea config

**AI →**

```
<𝒞=string:app:weather:korea>
/open ~/apps/weather/index.md
</𝒞>
```

```
<𝒞=string:app:weather:korea>
/set
{unit} = "celsius"
{default_city} = "Seoul"
{region} = "KR"
</𝒞>
```

### Japan config

**AI →**

```
<𝒞=string:app:weather:japan>
/open ~/apps/weather/index.md
</𝒞>
```

```
<𝒞=string:app:weather:japan>
/set
{unit} = "celsius"
{default_city} = "Tokyo"
{region} = "JP"
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

Same directory, different topics, different state. Each config's
navigation history and variables are fully independent.

> See [Topics](https://github.com/string-os/string/blob/main/docs/runtime/04-topics.md) for the `app:name:config` model.

---

## 11. Final directory structure

```
~/apps/weather/
├── index.md           # Home — search + current weather
│                        actions: search_city, current
│                        nav: main
│
├── forecast.md        # 7-Day forecast
│                        actions: forecast
│                        nav: main
│
├── alerts.md          # Alert management
│                        actions: create_alert, list_alerts
│                        nav: main
│
├── settings.md        # Configuration display
│                        nav: main
│
└── nav/
    └── main.md        # @home, @forecast, @alerts, @settings
```

Each page has focused actions. The nav menu appears on every page.
State flows across navigations within a topic.

---

## Single-page vs multi-page

| | Single-page | Multi-page |
|---|---|---|
| Files | 1 SFMD file | directory with multiple .md |
| "See forecast" | `/act.forecast` → template output | `/open @main.forecast` → new page |
| Navigation | none | `/open`, `/back`, `/nav` |
| Nav menu | not needed | `[!nav:main]` on every page |
| Best for | tools, dashboards, quick search | larger apps with sections |

The same weather API, two architectures. Choose based on how much
structure your app needs.

---

## Summary

| Pattern | How |
|---------|-----|
| **Navigation** | `[!nav:main](./nav/main.md)` on every page |
| **Actions** | `` ```act.name `` code blocks, hidden from AI |
| **Response templates** | `` ```act.name.response `` — format + store variables |
| **Page movement** | `/open @main.page` to navigate, `/back` to return |
| **Cross-page state** | Variables persist across navigations within a topic |
| **Config** | `/set` variables in a topic session |
| **Multi-config** | `app:name:config` — same directory, different topics |

The multi-page flow: **nav → pages → /open to move → /act to do → state flows across pages.**
`/open` to move, `/act` to do. Each page has focused responsibility.
