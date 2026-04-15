# String Cookbook

**Markdown is the content. `/command` is the interaction. Together they are an AI-native standard for exploring the web and using apps.**

`string` is the runtime for that standard — a browser for markdown that talks back, and the beginning of an OS built on the same idea.

---

## The whole idea, in one screen

An SFMD app is a markdown file (or several). It declares what it is and what it can do. A running `string` parses it, renders it for the reader, and dispatches commands against it.

A complete working app:

````markdown
---
title: Weather
name: weather
type: app
---

```act.now
CLI curl -s -G -d format=%l:+%C+%t+%w -d m https://wttr.in/{city}
  city: string (required) "City name"
```
````

Install it, then call its action:

```bash
$ string file:setup '/install --app ./apps/weather/index.md'
Installed app:weather

$ string app:weather '/open app:weather'
# Weather
A two-action weather app, backed by wttr.in...
[actions] /act.now --city <string> | ...

$ string app:weather '/act.now --city Seoul'
seoul: Sunny +20°C ↘6km/h
```

No server is running. No tool schema was registered. The markdown file declared an action block; `string` parsed it; the agent invoked it. That is the entire shape.

(The outputs above are lightly trimmed for readability — when you run the commands, each response is wrapped in a short framing envelope the CLI prints so callers can tell runtime output apart from noise. Chapter 00 shows it in full; chapter 03 explains how the client library parses it away so agent code never sees it.)

---

## Two ways an AI uses `string`

### Path 1 — the CLI, no code changes

Any agent that can run shell commands can use `string` today. The agent runs `string <topic> '<cmd>'` and reads the payload back from stdout. That's the whole integration:

```bash
$ string app:weather '/act.now --city Seoul'
<𝒞=string:app:weather>
seoul: Sunny +20°C ↘6km/h
</𝒞>
```

The `<𝒞=string:app:weather>` … `</𝒞>` pair is the default output framing — opening tag, payload, closing tag. The agent reads the payload between them. (A `--json` flag is also available for callers that strongly prefer a JSON envelope; the frame form is the default.)

Works in OpenClaw, Hermes Agent, or any framework with a shell tool. Zero integration work, zero code changes. This is how `string` should be used on first contact. Chapter [00](./00-weather.md) walks it through end to end with the weather app.

### Path 2 — the client library (recommended for agent builders)

If you're building an agent framework, embed [`@string-os/client`](https://www.npmjs.com/package/@string-os/client). Your code calls `string` by function, gets back structured results, and exposes **one tool** to the LLM that dispatches to every installed SFMD app — weather, calendar, wiki, anything.

See chapter [03 — Client library](./03-client-library.md) for the API, the single-tool pattern, and the daemon lifecycle.

---

## Chapters

| # | Chapter | What you'll see |
|---|---------|-----------------|
| 00 | [**Weather app, end to end**](./00-weather.md) | Install the app, drive it from a terminal, then watch an AI use it via the CLI. Start here. |
| 01 | [**Anatomy of an SFMD app**](./01-anatomy.md) | What the `weather` app is made of — frontmatter, action blocks, field schemas, shell-safe substitution — and how larger apps compose multiple files. |
| 02 | [**Why markdown + /command**](./02-compare.md) | The same weather capability built as an MCP server and as a function-calling tool, side by side with the SFMD version. |
| 03 | [**Client library**](./03-client-library.md) | Embedding `@string-os/client` in an agent framework. API reference and the single-tool pattern. |
| 04 | [**Porting Nano Banana Pro**](./04-porting-nano-banana-pro.md) | Worked example: take a real Codex skill (image generation via Gemini 3 Pro Image) and port it to a single-file SFMD app. The recipe transfers to any HTTP-based skill. |

Five chapters. The cookbook is deliberately narrow — its job is to make one thing viscerally clear: **the agent-facing interface is markdown plus a small set of commands, and most of the tools an AI needs fit that shape.**

---

## Who this is for

- **Developers building AI agents** in OpenClaw, Hermes Agent, or any framework with shell access or Node.js interop. You get a zero-integration tool today (Path 1) and a cleaner integration tomorrow (Path 2).
- **End users running those agents** who want to understand what their AI is actually doing. Chapter 01 is enough to read any SFMD app your agent installs.
- **Anyone evaluating a protocol for agent tools.** Chapter 02 is the comparison.

---

## The weather app

Every code path in this cookbook runs against one real app: [`apps/weather/index.md`](./apps/weather/index.md). It uses [wttr.in](https://wttr.in) (no API key), works the moment `string` installs it, and every output in the cookbook was captured from a live run. The file is 24 lines.

---

## Status: v0.1 preview

### Implemented today

- **Runtime:** parser, renderer, dispatcher, loader, env-store. [`string-os/string`](https://github.com/string-os/string).
- **Commands:** `/open`, `/nav`, `/back`, `/refresh`, `/close`, `/info`, `/ls`, `/source`, `/act`, `/tool`, `/exec`, `/write`, `/append`, `/edit`, `/verify`, `/undo`, `/set`, `/install`, `/uninstall`, `/session`, `/help`.
- **Topic types:** `file:`, `app:`, `web:`, `bash:`.
- **`string` CLI** — the zero-code integration path. Auto-starts the daemon on first use. Default output wraps payloads in `<𝒞=string:topic>` frames; `--json` flag is available for callers that prefer a JSON envelope.
- **`stringd` daemon** — long-lived process, loopback HTTP/SSE. See [`stringd-protocol-v0.1.md`](https://github.com/string-os/string/blob/main/docs/stringd-protocol-v0.1.md).
- **`@string-os/client`** — TypeScript client library, the recommended integration path for agent builders. ~185 lines, zero dependencies. See chapter 03.
- **SFMD core format** — frontmatter, action blocks, tool blocks, nav menus, shortcuts, includes. See [`string-os/sfmd`](https://github.com/string-os/sfmd).
- **`@string-os/compiler`** — multi-file SFMD composition via `[!include:]`.
- **HTML fallback** — any HTTPS URL can be opened; non-SFMD HTML is converted to markdown.
- **MCP bridge** (`@string-os/string-mcp`) — a third integration path for MCP hosts you don't control.

### Planned for v0.2 and beyond

- **`stringhub`** — a marketplace for installable SFMD apps.
- **Hosted reference sites** — [`string-os.org`](https://string-os.org), the company homepage, a shared wiki, all built as SFMD and browseable with `string`.
- **Signed packages** with provenance metadata, fine-grained capabilities, audit logging.
- **More language clients** — Python client in progress. Other languages welcome.
- **OS-scale features** — session persistence across devices, capability broker, multi-user scoping, skill search. These land as the runtime grows past "browser" toward "OS."
- **ChanFlow** — the multi-channel protocol `string`'s client library prototypes. To be released separately once it is ready to stand on its own.

The scope stays honest: what is implemented today is enough to run the full cookbook against real code.

## License

MIT. Copy any part of this cookbook into your own project without attribution.
