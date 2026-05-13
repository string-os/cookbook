# Weather for String 🌤️

Three-action weather app — current, forecast, and free-form location search. No API key, no signup.

## What you get

- **`@city-N` chaining.** `/act.search Springfield` returns 5 candidates as `@city-1..5`; then `/act.now @city-2` resolves to coordinates without copy-paste.
- **Prescriptive responses.** Every action ends with a `next:` line, so the agent always sees what's available next.
- **No-config install.** Backed by [wttr.in](https://wttr.in) and [Nominatim](https://nominatim.openstreetmap.org/) — both public, both free.

## Install

```
string main '/install --app ./weather/string.md'
string '/open app:weather'
```

That's it. No key, no setup.

## What it looks like

```
$ string app:weather '/act.now Seoul'

Seoul: Sunny +25°C →11km/h

next: /act.forecast Seoul for 3-day · /act.search "..." for ambiguous names
```

Ambiguous location? Search first:

```
$ string app:weather '/act.search Springfield'

Search: "Springfield"

- @city-1: Springfield, Sangamon County, Illinois, United States
- @city-2: Springfield, Hampden County, Massachusetts, United States
- @city-3: Springfield, Greene County, Missouri, United States
- @city-4: Springfield, Clark County, Ohio, United States
- @city-5: Springfield, Lane County, Oregon, United States

next: /act.now @city-N · /act.forecast @city-N

$ string app:weather '/act.now @city-3'

39.7990175,-89.6439575: Overcast +23°C
```

3-day forecast packs four daily timesteps + sun cycle into one compact block:

```
$ string app:weather '/act.forecast Seoul'

**Seoul, South Korea** — 3-day forecast (°C, km/h, mm rain)

### 2026-05-13 (15–26°C, avg 20°C)
- 06: 15°C, Sunny, wind WSW 3km/h, rain 0% (0.0mm)
- 12: 25°C, Sunny, wind W 7km/h, rain 0% (0.0mm)
- 18: 23°C, Partly Cloudy, wind W 10km/h, rain 0% (0.0mm)
- 21: 20°C, Clear, wind W 5km/h, rain 0% (0.0mm)
- sun 05:25 AM → 07:33 PM
### ...
```

## Actions

| Action | What |
|---|---|
| `/act.now <city>` | Current conditions, one line. Accepts city name or `lat,lon`. |
| `/act.forecast <city>` | 3-day forecast, 4 timesteps/day. |
| `/act.search <query>` | 5 candidates as `@city-N` shortcuts. Use first for ambiguous names. |

## Notes

- No `requirements.md` — there's nothing to configure.
- wttr.in is forgiving — it accepts city names, coordinates, airport codes, ZIPs.
- Multi-word cities: quote them or use `+` for spaces.

---

> 📘 **Cookbook reference.** A tight read-only app — three actions, no auth, no state. Demonstrates: `for:` loops on top-level array responses · `@city-N` shortcut binding from search results · `next:` discipline on every response.
