# Cross-Agent Portability

**The central pitch of SFMD in one cookbook chapter.** One SFMD file, multiple AI agents, semantically equivalent behavior, zero per-agent custom code.

This chapter walks you through loading a single `weather-dashboard.sfmd` file in three different AI agents and verifying they all (a) read the same facts and (b) invoke the same actions with the same arguments.

## What you'll need

- The String CLI installed locally (`npm install -g @string-os/string` or from source, see [`00-cli-quickstart.md`](./00-cli-quickstart.md))
- **Claude Desktop** with `@string-os/string-mcp` configured as an MCP server
- **One additional AI agent** that can either speak MCP (Cursor, any MCP-capable client) OR can read a markdown file you paste into its context (Gemini CLI, ChatGPT, a local Llama)

Total time: ~15 minutes.

## The file under test

Save this as `/tmp/weather-dashboard.sfmd`:

```markdown
# 🌤️ Weather Dashboard
**Location:** Suwon-si, Gyeonggi-do **Status:** Live (Last Updated: 14:19 KST)

### 🌡️ Current Conditions
- **Temperature:** 18°C (Feels like 17°C)
- **Weather:** Partly Cloudy ⛅
- **Humidity:** 45%
- **Wind:** 3 m/s (NW)

### 📌 Navigation
- [Hourly Forecast for today][@link_1]   (auto shortcut)
- [7-Day Extended Forecast][@link_2]
- [Saved city](@saved_city)              (named shortcut)
- [App configurations](./settings.md)    (relative path)

### ⚡ Quick Actions
**1. Search for a different city:** `/act.search_city --name "{City Name}"`
**2. Set a Custom Weather Alert:** `/act.create_alert --condition "{rain|snow|temp}"`
```

One file, three link styles, two typed actions, live-looking data. If the portability claim is real, all three agents should handle this the same way.

## The test prompts

You will ask each agent the same two questions:

1. **"What is the current temperature in Suwon, and what's it feel like?"**
2. **"Set up a rain alert for this dashboard."**

The pass condition is:

- **(a)** All three agents extract the same factual answer from the rendered content: "18°C, feels like 17°C".
- **(b)** For the second prompt, all three agents emit an action call equivalent to `/act.create_alert --condition "rain"`.

Byte-equality is not required. What matters is that the extracted facts match and the action invocation matches in name and argument values.

## Agent 1 — String CLI (reference)

```bash
string file:weather '/open /tmp/weather-dashboard.sfmd'
```

Expected output (wrapped in ChanFlow tags):

```
<𝒞=string:file:weather>
Opened /tmp/weather-dashboard.sfmd
---
# 🌤️ Weather Dashboard
**Location:** Suwon-si, Gyeonggi-do ...
(full rendered content)
</𝒞>
```

Then ask it to list actions:

```bash
string file:weather '/act'
```

```
<𝒞=string:file:weather>
Available actions:
  search_city  — Search for a different city
  create_alert — Set a Custom Weather Alert
</𝒞>
```

Run the rain-alert action:

```bash
string file:weather '/act.create_alert --condition rain'
```

Capture the full output of all three calls into `/tmp/portability-string-cli.txt`:

```bash
{
  string file:weather '/open /tmp/weather-dashboard.sfmd'
  string file:weather '/act'
  string file:weather '/act.create_alert --condition rain'
} > /tmp/portability-string-cli.txt 2>&1
```

## Agent 2 — Claude Desktop (via `string-mcp`)

**Prereq:** add this to your Claude Desktop MCP config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, equivalent path elsewhere):

```json
{
  "mcpServers": {
    "string": {
      "command": "npx",
      "args": ["@string-os/string-mcp"]
    }
  }
}
```

Restart Claude Desktop. You should see `string` in the MCP panel.

In a new Claude Desktop conversation, paste the following as a single user message:

> I have an SFMD weather dashboard file at `/tmp/weather-dashboard.sfmd`. Use the `string` MCP tool to open it, then tell me the current temperature in Suwon (and what it feels like), and then set up a rain alert for it.

Claude will call the `open` tool, read the rendered content, answer the factual question, and then call `act` with the `create_alert` action and `condition: rain` argument.

Capture the chat transcript (screenshot or copy-paste) to `/tmp/portability-claude-desktop.txt`.

## Agent 3 — Your choice

Pick one of these paths depending on what you have available:

### Option A — Another MCP-capable client (Cursor, Continue, etc.)

Same setup as Claude Desktop: register `@string-os/string-mcp` as an MCP server, then send the same prompt. Capture the transcript.

### Option B — Gemini CLI, ChatGPT, or any chat model

SFMD renders as plain Markdown — no MCP required. Paste the contents of `weather-dashboard.sfmd` directly into the model's context along with the test prompts:

> Here is an SFMD dashboard file. Read it and tell me the current temperature in Suwon and what it feels like. Then propose the exact action invocation to set up a rain alert, using the action syntax defined in the file.
>
> ```
> (paste file contents here)
> ```

The model should answer with "18°C, feels like 17°C" and propose something like `/act.create_alert --condition "rain"`.

Capture the transcript to `/tmp/portability-<agent>.txt`.

### Option C — Local Llama / other self-hosted model

Same as Option B. If the model has native tool-calling, you can also register a simple tool that wraps the String CLI, but for the v0.1 portability test, prompt-and-paste is sufficient.

## Verifying semantic equivalence

Compare the three captured outputs:

1. **Factual agreement** — does each output contain "18°C" or "18" for the current temperature, and "17°C" or "17" for the feels-like? ✅
2. **Action agreement** — does each output reference `create_alert` with a `rain` argument? ✅ (The exact wrapper syntax may differ: one agent says `/act.create_alert --condition rain`, another says `create_alert(condition="rain")`. That's fine. What matters is the action name and the argument value.)

If all three agree, the file passes the v0.1 portability test.

## Known limitations (v0.1)

- **Live data (`{{fetch:URL}}`)** is marked experimental in the spec and may render differently across agents depending on whether the runtime substitutes it before the agent sees it or leaves it as a placeholder. For the portability test, the Weather Dashboard file above uses static values to avoid this variability.
- **Auto shortcuts (`[@link_1]`)** are a navigation hint for runtimes that support shortcut expansion. Agents reading raw markdown will see them as plain link references, which is acceptable — the pass condition is factual agreement on the rendered content, not identical hyperlink handling.
- **Action execution vs action proposal.** Agent 1 (String CLI) actually executes `/act.create_alert`. Agents 2 and 3 propose it. Both count as passing the portability test, because the point is that the action contract is the same across agents.

## If a test fails

- **Factual mismatch** — the agent hallucinated data or failed to read the rendered content. File an issue at <https://github.com/string-os/cookbook/issues> with the full prompt and response.
- **Action mismatch** — the agent invented a different action or different arguments. This is a more interesting failure mode. Please file an issue with the transcript.
- **Runtime error in the String CLI** — file an issue at <https://github.com/string-os/string/issues> with the `string --help` version and the error message.

## Why this matters

This is the concrete, reproducible test of the SFMD thesis. If you can get three different agents to read the same file and agree on facts and actions, the "portable agent surface" claim is real. If you can't, we need to know — please open an issue.
