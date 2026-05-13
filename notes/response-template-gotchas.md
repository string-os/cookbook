# Response template gotchas

Three traps that bit us while building `moltbook`, `weather`, and `nano-banana-pro`. Each is small in isolation but easy to miss until the output looks wrong. Recorded so future authors don't repeat the discovery.

---

## 1. Input fields shadow assignment vars

**Symptom.** You assign `{city} = {Response.body.area.name}` to extract a clean name from the response, then reference `{city}` in a later output line — and you get the *original input value* instead of the resolved name.

**Example (weather):**

```
GET https://wttr.in/{city}?format=j1
  city: string (required) "City name or lat,lon"

# Response template:
{city} = {Response.body.nearest_area[0].areaName[0].value}
**{city}** — 3-day forecast
```

Caller runs `/act.forecast --city 37.20,-93.29` (passing coords). The template assigns `{city} = "Springfield"` from the response. But the output renders `**37.20,-93.29** — 3-day forecast` — because the input field `{city}` wins the substitution pass.

**Why.** Output lines do three substitution passes: `Response.body.X` → session vars → action payload fields. The payload pass runs last, so a name that collides with an input field gets overwritten.

**Fix.** Use a different name for the assignment var. Convention: name input fields after the agent-facing concept (`city`, `name`, `id`), and name response-derived vars after the *data shape* (`area`, `display_name`, `submolt`, `aname`).

```
{area} = {Response.body.nearest_area[0].areaName[0].value}
**{area}** — 3-day forecast
```

---

## 2. `{@shortcut}` in output doesn't substitute — it's the shortcut label

**Symptom.** You write `{@image} = {filename}` then a display line `Saved: {@image}`. The output shows literally `Saved: @image`, not `Saved: /tmp/crane.jpg`.

**Why.** `{@name}` is *not* a variable reference — it's the shortcut placeholder. In list contexts (`- {@post}: {p.title}`) the renderer turns it into the auto-enumerated tag `@post-1`, `@post-2`, etc. In a singleton context, it renders the bare label. Either way, it does not expand to the assigned value.

**The rule:** shortcuts are for *chaining*, not for *display*. Bind once, then reference the value through a normal field or var.

**Fix.** Use the input field (or a normal session var) for display, keep the shortcut for the `next:` line:

```
{@image} = {filename}
Saved: {filename} ({mime}) — also bound as @image

next: /act.edit -i @image -f <new>
```

The shortcut `@image` only needs to resolve when an *action* receives it as an argument (the runtime resolves shortcuts at call time). Display lines should never reference `{@name}` expecting the value.

---

## 3. `for:` loops over primitive arrays ~~drop the value~~ — FIXED in 2026-05-13 runtime

**Was a problem.** Iterating a string array used to render literal `{var}` because the body substitution regex only matched `{var.field}` (with a dot).

**Now works.** A second pass handles bare `{var}` for primitive iteration:

```
for: t in Response.body.what_to_do_next
- {t}
end:
```

renders one bullet per string element. Object iteration (`{p.title}`) is unaffected.

**Runtime change.** `packages/string/src/commands/helpers.ts` — added a second `.replace` after the `{itemVar.field}` pass that handles bare `{itemVar}` and stringifies the current iteration value.

---

## Authoring checklist

When writing a response template, scan for these before shipping:

- [ ] Every assigned var name (`{x} = ...`) is **different** from every input field name. Collision = silent shadowing.
- [ ] Display lines reference `{field}` or `{var}`, never `{@shortcut}`.
- [ ] Shortcut references appear in `next:` lines and action arguments, not in prose.
- [ ] `for:` loops iterate object arrays. If the array is primitives, emit the whole array as `{Response.body.path}` and move on.
- [ ] Every response ends with a `next:` line (no dead ends). At least one suggestion per response.

---

## Provenance

Discovered:

- 2026-05-13 — moltbook `act.home.response` (gotcha 3 — `what_to_do_next` printed `{t}` four times) — **fixed in runtime same day**
- 2026-05-13 — weather `act.forecast.response` (gotcha 1 — `{city}` collided with input field)
- 2026-05-13 — nano-banana-pro `act.generate.response` (gotcha 2 — `Saved: @image` instead of path)

All three apps now demonstrate the fixes. See [`apps/moltbook/string.md`](../apps/moltbook/string.md), [`apps/weather/string.md`](../apps/weather/string.md), [`apps/nano-banana-pro/string.md`](../apps/nano-banana-pro/string.md).

## Related runtime change (2026-05-13)

**`$VAR` resolution in field defaults.** Authors can now write:

```
GET https://api.example.com/{city}?...
  city: string "City" = "$CITY"
```

The runtime resolves `$CITY` from the env store *before* URL-encoding, so config-scoped env vars feed naturally into URLs. If `$CITY` is unset, the action fails with a clear `Unresolved environment variable in field default` error (rather than silently passing the literal `$CITY` through to URL encoding).

This is what makes `/open app:weather:seoul` work as a one-touch dashboard after `/set $CITY = "seoul"` in that config scope.

**Runtime change.** `packages/string/src/commands/action.ts` — after default-fill, resolve `$VAR` refs in string payload values; collect any that fail to resolve and return `INVALID_PAYLOAD` with a "set it with `/set $X=...`" hint.
