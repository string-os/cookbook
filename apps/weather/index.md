---
title: Weather
name: weather
type: app
version: 0.1.0
---

# Weather

A two-action weather app, backed by [wttr.in](https://wttr.in) — no API
key, no signup, no server to run. Works the moment it is installed.

## Actions

- `/act.now --city <name>` — current conditions, one line
- `/act.forecast --city <name>` — detailed forecast with wind and humidity

For multi-word cities, use `+` in place of spaces: `--city New+York`.

```act.now
CLI curl -s --max-time 10 -G -d format=%l:+%C+%t+%w -d m https://wttr.in/{city}
  city: string (required) "City name"
```

```act.forecast
CLI curl -s --max-time 10 -G -d format=%l:+%C+%t+%w+%h+%p -d m https://wttr.in/{city}
  city: string (required) "City name"
```
