# Cookbook: CLI App — Weather

Build a real SFMD app that wraps an external API, install it,
and use it from the CLI topic mode. No API key required.

**What you'll learn:**
- Writing a CLI-action SFMD app (curl wrapper)
- Installing and registering an app
- Using topic mode: `string app:weather '/act.now --city Seoul'`
- URI path parameters `{city}` with session variable fallback

> See [Actions](https://github.com/string-os/string/blob/main/docs/runtime/05-actions.md) for action syntax.
> See [Topics](https://github.com/string-os/string/blob/main/docs/runtime/04-topics.md) for the session model.

---

## Why wrap an API in an SFMD app?

The AI doesn't need to remember API endpoints, format strings, or auth headers.
The app author encodes that knowledge once. The AI just calls actions:

```bash
# Without String — AI must know the API
curl -s "wttr.in/Seoul?format=%l:+%c+%t+(feels+%f)+%h+%w"

# With String — AI calls a named action
string app:weather '/act.now --city Seoul'
```

Same result, but the second version is:
- **Self-documenting**: `/open weather` lists all actions and their parameters
- **Session-aware**: the daemon keeps state between calls
- **Installable**: the user sets it up once, every AI session can use it

---

## 1. Write the app

One file. Three actions. No response templates — raw CLI output goes
straight to the AI.

`~/.string/packages/weather/index.md`:

````markdown
---
name: weather
type: app
description: Weather conditions and forecasts powered by wttr.in
---

# Weather

Current conditions and forecasts for any city. No API key required.

## Quick Check

```act.now
CLI curl -s "wttr.in/{city}?format=%l:+%c+%t+(feels+%f)+%h+%w"
  city: string (required) "City name (e.g. Seoul, London, New+York)"
```

## Today

```act.today
CLI curl -s "wttr.in/{city}?0T"
  city: string (required) "City name"
```

## 3-Day Forecast

```act.forecast
CLI curl -s "wttr.in/{city}?T"
  city: string (required) "City name"
```
````

**Key decisions:**

- **CLI, not GET**: wttr.in returns plain text, not JSON. CLI curl gives
  us the output as-is. No response template needed.
- **`{city}` in the URL**: this is a URI path parameter. When the AI calls
  `--city Seoul`, String replaces `{city}` with `Seoul` before executing.
  If `{city}` is also a session variable, it's used as fallback when
  `--city` is omitted.
- **No `default` action**: the app describes itself on `/open`.
  The AI reads the actions list and picks what to call.

---

## 2. Install the app

Create the package directory and register it:

```bash
mkdir -p ~/.string/packages/weather
# (copy the file above to ~/.string/packages/weather/index.md)
```

Register in `~/.string/config.json`:

```json
{
  "apps": {
    "weather": "file:///home/you/.string/packages/weather/index.md"
  }
}
```

Or use the built-in installer from a String session:

```
/install --app ./weather.md
```

---

## 3. Use it

### Open — see what's available

```bash
string app:weather '/open weather'
```

Output:
```
Opened .string/packages/weather/index.md
---
[actions] /act.now --city <string> | /act.today --city <string> | /act.forecast --city <string>
          /act.<name> --help for details

# Weather

Current conditions and forecasts for any city. No API key required.
```

The AI now knows exactly what actions exist and what parameters they take.

### Current conditions

```bash
string app:weather '/act.now --city Seoul'
```

```
seoul: ☀️  +9°C (feels +10°C) 62% ↗4km/h
```

### Today's detail

```bash
string app:weather '/act.today --city London'
```

```
Weather report: london

       .-.      Rain shower
      (   ).    +6(2) °C
     (___(__)   → 26 km/h
    ‚'‚'‚'‚'    10 km
    ‚'‚'‚'‚'    0.1 mm
```

### 3-day forecast

```bash
string app:weather '/act.forecast --city Tokyo'
```

Full ASCII art weather table with morning/noon/evening/night.

### JSON output (for programmatic use)

```bash
string --json app:weather '/act.now --city Seoul'
```

```json
{"ok":true,"code":null,"content":"seoul: ☀️  +9°C (feels +10°C) 62% ↗4km/h"}
```

---

## 4. Session persistence

Each `app:weather` call goes through the same daemon session.
Session variables set via `/set` persist across calls:

```bash
string app:weather '/set {city} = "Seoul"'
string app:weather '/act.now --city {city}'
```

The second call uses the session variable.
Different configs get isolated sessions:

```bash
string app:weather:home '/set {city} = "Seoul"'
string app:weather:office '/set {city} = "London"'
```

---

## Design notes

### CLI vs GET actions

| | CLI | GET |
|---|---|---|
| **Output** | Raw stdout | `{Response.body}` (JSON parsed) |
| **Response template** | Not useful (`jsonBody` is null) | Full extraction: `{var} = {Response.body.field}` |
| **Best for** | Text APIs, shell tools | JSON APIs, variable chaining |

Use CLI when the API returns human-readable text (wttr.in, man pages, ASCII tools).
Use GET when you need to extract fields and chain variables across actions.

### URI path parameters

`{name}` in the action URI is resolved in this order:
1. `--name` flag value (from the invocation)
2. `{name}` session variable (fallback)
3. Left as `{name}` literal (will likely cause an error)

This means an AI can `/set {city} = "Seoul"` once, then call
`/act.now` and `/act.forecast` without repeating `--city` — if the
action's city field is optional.

### App vs bare curl

The app author understands the API. The AI user doesn't have to.
This is the core value: **encode API knowledge as actions,
expose clean interfaces to the AI.**
