# 01 — Anatomy of an SFMD app

**Goal:** understand every line of `apps/weather/index.md`, then see how larger apps compose multiple files. By the end you can read any SFMD app your agent installs and write your own.

This is reference material. The previous chapter showed what the weather app *does*; this one shows what it *is*.

---

## The minimal app

[`apps/weather/index.md`](./apps/weather/index.md) in full:

````markdown
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
````

24 lines, one file, no build step. This is **the minimum** — the smallest shape an SFMD app can take and still be useful. A real app can be bigger, and the last section of this chapter shows how. Start with the minimum because it is what everything else builds on.

---

## Frontmatter (lines 1–6)

```yaml
---
title: Weather
name: weather
type: app
version: 0.1.0
---
```

YAML at the top of the file. Four keys matter to the runtime:

| Key | Purpose |
|-----|---------|
| `title` | Shown by `/info`. Used by agents when they describe what they're looking at. |
| `name` | The package name when the file is installed. Must match `[a-zA-Z0-9_-]+` — the installer sanitizes it. This is how the file becomes addressable as a bare `weather`. |
| `type` | `app` or `tool`. Tells the installer which registry to use — `/open app:weather` works for apps, `/tool:weather` for tools. |
| `version` | SemVer string. Surfaced by `/info` so agents can check which version they're looking at. |

Anything else you add to the frontmatter is preserved but ignored by the runtime. Use custom keys for your own tooling without fear of collision.

---

## Body (lines 7–17)

```markdown
# Weather

A two-action weather app, backed by [wttr.in](https://wttr.in) — no API
key, no signup, no server to run. Works the moment it is installed.

## Actions

- `/act.now --city <name>` — current conditions, one line
- `/act.forecast --city <name>` — detailed forecast with wind and humidity

For multi-word cities, use `+` in place of spaces: `--city New+York`.
```

Plain CommonMark. Anything you can write in a regular markdown file works here — headings, lists, emphasis, links, code spans, fenced code blocks, tables. The renderer strips SFMD-specific blocks (action blocks, nav directives, include directives) but leaves everything else alone.

**The body is what the agent reads as prose.** Write it for an agent that has already opened the app and wants to know how to use it — one or two paragraphs of context, a quick reference for the actions, any gotchas. The bulleted `/act.now --city <name>` lines exist for the agent to read, not for the runtime to parse.

The link to `wttr.in` turns into an auto-assigned shortcut (`@link-1`) so the URL stays out of the agent's working context. `/nav page` lists it. For a one-file app this is overkill; for a page with many links it is what keeps the viewport short.

---

## Action blocks (lines 19–27)

Two fenced code blocks with the language tag `act.<name>`. Each block declares one invocable action.

````
```act.now
CLI curl -s --max-time 10 -G -d format=%l:+%C+%t+%w -d m https://wttr.in/{city}
  city: string (required) "City name"
```
````

Three parts:

### 1. The block header

````
```act.now
````

The fence language is `act.<id>` where `<id>` is the action name. IDs must match `[a-zA-Z0-9_-]+`. The caller invokes it as `/act.now`.

### 2. The invocation template

```
CLI curl -s --max-time 10 -G -d format=%l:+%C+%t+%w -d m https://wttr.in/{city}
```

The first line of the block is `<METHOD> <template>`.

- **Method** is one of `CLI`, `GET`, `POST`, `PUT`, `PATCH`, `DELETE`. `CLI` runs the template as a shell command; HTTP methods use the template as a URL and fetch it.
- **Template** is the command (for CLI) or URL (for HTTP). Use `{field}` to substitute a field value into the template.

This action's template is a `curl` command with flags. `curl -G -d foo -d bar URL` puts `foo` and `bar` into the query string of `URL`, producing `URL?foo&bar`. That construction is important: it is what keeps a literal `&` out of the template string. If a template had `URL?foo&bar` directly, bash would interpret the `&` as "background the previous command and then run `bar`." This app author sidestepped the whole category by letting `curl` assemble the query string.

### 3. The field list

```
  city: string (required) "City name"
```

Every indented line below the template is one field. The syntax is:

```
  <name>: <type> [(required|optional)] ["description"] [= "default"]
```

- `<name>`: the flag the caller passes. `city` becomes `--city` on the command line.
- `<type>`: `string`, `number`, or `boolean`. Used for validation.
- `(required)` / `(optional)`: whether the caller must provide this flag.
- `"description"`: human-readable hint. Shown in the `/act` listing and in `--help`.
- `= "default"`: default value if the caller omits the flag.

The weather app has one field per action — `city`, required, no default. A missing `--city` flag returns a structured error and never touches the shell.

---

## How `{city}` becomes safe

When you call `/act.now --city Seoul`, the runtime does four things in order:

1. **Parse the flag string.** `--city Seoul` becomes the payload `{ city: "Seoul" }`.
2. **Validate against the field list.** `city` is required; it is present; done. A missing required field returns an error and never runs the template.
3. **Substitute into the template.** Every `{field}` is replaced with the value the caller provided, **and** the value is shell-quoted first. `Seoul` has no metacharacters so it stays as `Seoul`. `; rm -rf /` would become `'; rm -rf /'` — wrapped in single quotes, which bash parses as a literal string, not as a command separator.
4. **Run.** For `CLI`, the substituted template is passed to `/bin/bash -c`. For HTTP, the substituted URL is fetched, and field values are URL-encoded instead of shell-quoted.

The template author does not write quoting logic. The runtime does it once, the same way, for every SFMD action ever written. The one rule: **the template must not wrap `{field}` in outer quotes**, because the slot self-quotes. Same convention as bash's `"$@"`.

---

## What the agent sees vs what's hidden

When an agent inspects the action via `/act`, `/act.<name> --help`, or the `[actions]` hint at the top of `/open`, it gets exactly the **call interface**: the verb name, each field's type and required-ness, and the description. Nothing more.

```
/act.now
   --city <string> (required) — City name
```

It does **not** see:

- the underlying method (`CLI` vs `POST` vs `GET` etc.)
- the URL or the bash template
- the request `body:` template, headers, or `$VAR` references
- the response-template `save:` / `decode:` / `to:` directives

Those are *implementation* — the runtime's job, not the agent's. Hiding them keeps the help surface focused on "what to call, what to pass" and saves prompt tokens on every call.

When you genuinely need to inspect what's behind an action — security audit, debugging an unexpected response, curiosity about the underlying API — `/source` dumps the raw `.md` file, including frontmatter, body, action blocks, and response templates:

```bash
string app:weather '/source'
```

`/source` is the escape hatch. Reach for it when you need to see the implementation; otherwise let the runtime do its job.

---

## Response handling

For `CLI` actions, stdout becomes the action result. Stderr is appended. A non-zero exit code surfaces as an error with a code like `EXIT_1`.

For HTTP actions, the response body is parsed as an SFMD document, rendered, and set as the current document. The agent sees the response as a viewport it can `/nav` and `/act` on. This is how an SFMD site responds to an action by loading a new page — you POST a form, you land on the result page, you navigate from there.

---

## Bigger apps: multiple files

An SFMD app is **not** required to be one file. The weather app happens to be, because one file is the minimum. Larger apps compose several files using three mechanisms.

### Nav files

A nav file is a markdown file whose only job is to list shortcuts:

```markdown
<!-- nav/main.md -->
[@home Home](../index.md)
[@docs Documentation](../docs/index.md)
[@api API Reference](../docs/api.md)
```

Any page that wants this navigation references it with a directive:

```markdown
[!nav:main](./nav/main.md)
```

At load time, the runtime merges the shortcuts into the current session under the menu name `main`. `/nav main` lists them. `/open @main.home` follows one. The menu lives outside the page, so every page in the app can share the same nav without duplicating entries.

### Includes

Includes are the compiler-level way to compose one document out of many:

```markdown
# Home

[!include:intro](./sections/intro.md)

[!include:pricing](./sections/pricing.md)
```

`@string-os/compiler` reads this skeleton, resolves each `[!include:id](path)` directive, and produces a single output file with each block inlined and wrapped in `<!-- #id -->` / `<!-- /id -->` markers. The resulting file is still a normal SFMD page. The runtime does not need to know about includes — the compiler does.

Use includes when a page has several distinct sections that want their own source files, or when the same section needs to appear on multiple pages.

### Tool blocks

A `tool:<name>` block bundles several actions that share context — a base URL, authentication headers, environment variables:

````markdown
```tool:weather
GET https://api.openweathermap.org/data/2.5
  headers:
    Authorization: Bearer $WEATHER_TOKEN

  ```act.current
  GET /weather?q={city}
    city: string (required)
  ```

  ```act.forecast
  GET /forecast?q={city}&cnt={days}
    city: string (required)
    days: number = "5"
  ```
```
````

Invoking is `/tool:weather.current --city Seoul` or `/tool:weather.forecast --city Seoul --days 7`. Both inherit the base URL and auth header from the enclosing tool block. The weather app in this cookbook does not use a tool block because it has no shared context — two independent `act.` blocks are simpler.

---

## What the file does not need

Worth stating explicitly, because the absences are part of the point:

- **No tool schema.** The field list *is* the schema. The runtime parses it at open time.
- **No input validation code.** The `(required)` marker and type declaration are the validation.
- **No server.** `CLI` actions run under the daemon's shell. `HTTP` actions use the runtime's fetch path. There is nothing for the app author to deploy.
- **No auth scaffolding.** If an action needs a token, it reads `$ENV_VAR`, and the user sets it with `/set VAR value`. Scoped per-user by the env-store.
- **No tests.** The cookbook doesn't ship any. For the weather app, three `/act` calls from the terminal are the test. For larger apps, you write whatever test suite suits you — `string` is a normal Node package.

---

## Next

- **[02 — Why markdown + /command](./02-compare.md)** — the honest comparison with MCP servers and function calling.
