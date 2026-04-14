# Cookbook: Document Editing

A complete walkthrough of creating, editing, and versioning documents
through String. Every step is shown as a real AI↔String conversation
using ChanFlow channel tags.

**Scenario:** Create meeting notes, edit them, recover from a mistake,
and manage versions — first as a simple one-page document, then as a
structured document with includes.

→ See [Editing](https://github.com/string-os/string/blob/main/docs/runtime/09-editing.md) for the full command reference.
→ See [Authoring](https://github.com/string-os/string/blob/main/docs/runtime/08-authoring.md) for document structure.

---

## Part 1: Simple document (plain Markdown)

Most documents don't need any SFMD features. Just write Markdown.

### 1.1 Create a document

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
# Sprint Review — 2026-03-19

## Attendees
- Alice (PM)
- Bob (Engineering)
- Carol (Design)

## Decisions
(to be filled during meeting)

## Action Items
- [ ] Alice: finalize roadmap
- [ ] Bob: deploy staging fix

## Notes
(live notes)
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md>
✓ created ~/notes/meeting-0319.md (18 lines)
</𝒞>
```

Plain Markdown. No frontmatter, no block markers. Any Markdown viewer
can render this — GitHub, Obsidian, VS Code.

Content without a `/command` prefix is treated as a write — equivalent
to explicit `/write`:

```
<𝒞=string:~/notes/meeting-0319.md>
/write
# Sprint Review — 2026-03-19
...
</𝒞>
```

Both forms are identical. Bare content is preferred for brevity.

### Overwrite feedback

If the file already exists, sending content overwrites it. String
tells the AI what happened and offers a way back:

```
<𝒞=string:~/notes/meeting-0319.md>
✓ overwritten ~/notes/meeting-0319.md (18 lines, was 12 lines)
  /undo to revert
</𝒞>
```

### 1.2 View raw source with /edit

Before modifying, the AI opens edit mode to see line numbers.

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/edit
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md mode=edit>
  1 │ # Sprint Review — 2026-03-19
  2 │
  3 │ ## Attendees
  4 │ - Alice (PM)
  5 │ - Bob (Engineering)
  6 │ - Carol (Design)
  7 │
  8 │ ## Decisions
  9 │ (to be filled during meeting)
 10 │
 11 │ ## Action Items
 12 │ - [ ] Alice: finalize roadmap
 13 │ - [ ] Bob: deploy staging fix
 14 │
 15 │ ## Notes
 16 │ (live notes)
</𝒞>
```

### 1.3 Edit by line number

The meeting produced decisions. The AI replaces the placeholder.

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/replace :L9
1. Ship v2.1 by Friday — no scope changes
2. Move design review to biweekly
3. Alice owns stakeholder comms going forward
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md mode=edit>
✓ ~/notes/meeting-0319.md:L9 — Added 3 lines, removed 1 line

  8  │ ## Decisions
  9 -│ (to be filled during meeting)
  9 +│ 1. Ship v2.1 by Friday — no scope changes
 10 +│ 2. Move design review to biweekly
 11 +│ 3. Alice owns stakeholder comms going forward
 12  │
</𝒞>
```

One line replaced with three. The diff feedback shows exactly what
changed.

### 1.4 Edit a line range

The AI needs fresh line numbers after the previous edit shifted lines.

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/edit
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md mode=edit>
  1 │ # Sprint Review — 2026-03-19
  2 │
  3 │ ## Attendees
  4 │ - Alice (PM)
  5 │ - Bob (Engineering)
  6 │ - Carol (Design)
  7 │
  8 │ ## Decisions
  9 │ 1. Ship v2.1 by Friday — no scope changes
 10 │ 2. Move design review to biweekly
 11 │ 3. Alice owns stakeholder comms going forward
 12 │
 13 │ ## Action Items
 14 │ - [ ] Alice: finalize roadmap
 15 │ - [ ] Bob: deploy staging fix
 16 │
 17 │ ## Notes
 18 │ (live notes)
</𝒞>
```

Now update both action items.

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/replace :L14-L15
- [ ] Alice: finalize roadmap by EOD Thursday
- [x] Bob: deploy staging fix (done during meeting)
- [ ] Carol: update design system tokens
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md mode=edit>
✓ ~/notes/meeting-0319.md:L14-L15 — Added 3 lines, removed 2 lines

 13  │ ## Action Items
 14 -│ - [ ] Alice: finalize roadmap
 15 -│ - [ ] Bob: deploy staging fix
 14 +│ - [ ] Alice: finalize roadmap by EOD Thursday
 15 +│ - [x] Bob: deploy staging fix (done during meeting)
 16 +│ - [ ] Carol: update design system tokens
 17  │
</𝒞>
```

Line numbers shift after every edit — always `/edit` again before
the next line-level replace.

### 1.5 Recover from a mistake with /undo

The AI accidentally overwrites the wrong section.

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/replace :L18
Wrong notes pasted here by mistake.
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md mode=edit>
✓ ~/notes/meeting-0319.md:L18 — Added 1 line, removed 1 line

 17  │ ## Notes
 18 -│ (live notes)
 18 +│ Wrong notes pasted here by mistake.
</𝒞>
```

Oops. Undo it.

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/undo
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md mode=edit>
✓ ~/notes/meeting-0319.md — Reverted last change

 17  │ ## Notes
 18 -│ Wrong notes pasted here by mistake.
 18 +│ (live notes)
</𝒞>
```

Restored. `/undo` holds exactly one state — use it immediately.

### 1.6 Fill in the notes and verify

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/replace :L18
- v2.1 scope is locked. No new features until after release.
- Staging deploy succeeded during meeting — Bob confirmed.
- Design review cadence changing because Carol's team is growing.
- Alice will send stakeholder update email by Thursday.
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md mode=edit>
✓ ~/notes/meeting-0319.md:L18 — Added 4 lines, removed 1 line

 17  │ ## Notes
 18 -│ (live notes)
 18 +│ - v2.1 scope is locked. No new features until after release.
 19 +│ - Staging deploy succeeded during meeting — Bob confirmed.
 20 +│ - Design review cadence changing because Carol's team is growing.
 21 +│ - Alice will send stakeholder update email by Thursday.
</𝒞>
```

Verify the final state:

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/refresh
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md>
# Sprint Review — 2026-03-19

## Attendees
- Alice (PM)
- Bob (Engineering)
- Carol (Design)

## Decisions
1. Ship v2.1 by Friday — no scope changes
2. Move design review to biweekly
3. Alice owns stakeholder comms going forward

## Action Items
- [ ] Alice: finalize roadmap by EOD Thursday
- [x] Bob: deploy staging fix (done during meeting)
- [ ] Carol: update design system tokens

## Notes
- v2.1 scope is locked. No new features until after release.
- Staging deploy succeeded during meeting — Bob confirmed.
- Design review cadence changing because Carol's team is growing.
- Alice will send stakeholder update email by Thursday.
</𝒞>
```

Clean rendered view. The document reads like a finished meeting record.

### 1.7 Version control

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/commit --message "Sprint review notes complete"
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md>
✓ committed ~/notes/meeting-0319.md
  c1  2026-03-19 15:00  "Sprint review notes complete"
</𝒞>
```

Add a follow-up section, commit again:

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/append

## Follow-up
Next sprint review: 2026-03-26, same time.
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md mode=edit>
✓ ~/notes/meeting-0319.md — Appended 3 lines (24 total)

 21  │ - Alice will send stakeholder update email by Thursday.
 22  │
 23 +│ ## Follow-up
 24 +│ Next sprint review: 2026-03-26, same time.
</𝒞>
```

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/commit --message "Added follow-up section"
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md>
✓ committed ~/notes/meeting-0319.md
  c2  2026-03-19 15:05  "Added follow-up section"
</𝒞>
```

### View history and time travel

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/log
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md>
c2  2026-03-19 15:05  "Added follow-up section"
c1  2026-03-19 15:00  "Sprint review notes complete"
</𝒞>
```

**AI →**

```
<𝒞=string:~/notes/meeting-0319.md>
/open @c1
</𝒞>
```

**String →**

```
<𝒞=string:~/notes/meeting-0319.md>
# Sprint Review — 2026-03-19

## Attendees
- Alice (PM)
- Bob (Engineering)
- Carol (Design)

## Decisions
1. Ship v2.1 by Friday — no scope changes
2. Move design review to biweekly
3. Alice owns stakeholder comms going forward

## Action Items
- [ ] Alice: finalize roadmap by EOD Thursday
- [x] Bob: deploy staging fix (done during meeting)
- [ ] Carol: update design system tokens

## Notes
- v2.1 scope is locked. No new features until after release.
- Staging deploy succeeded during meeting — Bob confirmed.
- Design review cadence changing because Carol's team is growing.
- Alice will send stakeholder update email by Thursday.
</𝒞>
```

Read-only snapshot at `c1` — before the follow-up section. To restore
it, the AI would send the content back to the same topic.

---

## Part 2: Structured document (with includes)

When a document grows large, split it into pieces using `!include`
directives. Each include becomes an addressable block that the AI
can edit independently.

→ See [Authoring — Including content](https://github.com/string-os/string/blob/main/docs/runtime/08-authoring.md) for include syntax.

### 2.1 Create the main document

The main file is a skeleton with includes. Each `[!include:id]()` pulls
content from `meeting-0319.source/id.md` automatically.

**AI →**

```
<𝒞=string:~/docs/meeting-0319.md>
# Sprint Review — 2026-03-19

## Attendees
- Alice (PM)
- Bob (Engineering)
- Carol (Design)

[!include:decisions]()

[!include:action-items]()

[!include:notes]()
</𝒞>
```

**String →**

```
<𝒞=string:~/docs/meeting-0319.md>
✓ created ~/docs/meeting-0319.md (12 lines)
</𝒞>
```

The attendees section is short enough to live inline. The three longer
sections are includes — they'll be separate files under
`meeting-0319.source/`.

### 2.2 Write to the included blocks

Targeting `#block` in the channel tag writes to the source file for
that include. Content without a `/command` prefix is treated as `/write`.

**AI →**

```
<𝒞=string:~/docs/meeting-0319.md#decisions>
## Decisions
1. Ship v2.1 by Friday — no scope changes
2. Move design review to biweekly
3. Alice owns stakeholder comms going forward
</𝒞>
```

**String →**

```
<𝒞=string:~/docs/meeting-0319.md>
✓ created ~/docs/meeting-0319.source/decisions.md (4 lines)
</𝒞>
```

**AI →**

```
<𝒞=string:~/docs/meeting-0319.md#action-items>
## Action Items
- [ ] Alice: finalize roadmap by EOD Thursday
- [ ] Bob: deploy staging fix
- [ ] Carol: update design system tokens
</𝒞>
```

**String →**

```
<𝒞=string:~/docs/meeting-0319.md>
✓ created ~/docs/meeting-0319.source/action-items.md (4 lines)
</𝒞>
```

**AI →**

```
<𝒞=string:~/docs/meeting-0319.md#notes>
## Notes
- v2.1 scope is locked. No new features until after release.
- Staging deploy succeeded during meeting — Bob confirmed.
</𝒞>
```

**String →**

```
<𝒞=string:~/docs/meeting-0319.md>
✓ created ~/docs/meeting-0319.source/notes.md (3 lines)
</𝒞>
```

The directory structure now:

```
~/docs/
├── meeting-0319.md                  # main document
└── meeting-0319.source/
    ├── decisions.md                 # #decisions block
    ├── action-items.md              # #action-items block
    └── notes.md                     # #notes block
```

### 2.3 Read the full assembled document

**AI →**

```
<𝒞=string:~/docs/meeting-0319.md>
/open
</𝒞>
```

**String →**

```
<𝒞=string:~/docs/meeting-0319.md>
# Sprint Review — 2026-03-19

## Attendees
- Alice (PM)
- Bob (Engineering)
- Carol (Design)

## Decisions
1. Ship v2.1 by Friday — no scope changes
2. Move design review to biweekly
3. Alice owns stakeholder comms going forward

## Action Items
- [ ] Alice: finalize roadmap by EOD Thursday
- [ ] Bob: deploy staging fix
- [ ] Carol: update design system tokens

## Notes
- v2.1 scope is locked. No new features until after release.
- Staging deploy succeeded during meeting — Bob confirmed.
</𝒞>
```

String assembled all includes into one seamless document. The AI sees
clean Markdown — no include directives, no block markers.

### 2.4 Read a single block

**AI →**

```
<𝒞=string:~/docs/meeting-0319.md>
/open #decisions
</𝒞>
```

**String →**

```
<𝒞=string:~/docs/meeting-0319.md>
## Decisions
1. Ship v2.1 by Friday — no scope changes
2. Move design review to biweekly
3. Alice owns stakeholder comms going forward
</𝒞>
```

Token-efficient — loads just the section the AI needs.

### 2.5 Update a block

To update a block, write to it again — same as creating it. The
block in the topic (`#decisions`) determines where the content goes.

**AI →**

```
<𝒞=string:~/docs/meeting-0319.md#decisions>
## Decisions
1. Ship v2.1 by Friday — no scope changes
2. Move design review to biweekly
3. Alice owns stakeholder comms going forward
4. Adopt new CI pipeline starting next sprint
</𝒞>
```

**String →**

```
<𝒞=string:~/docs/meeting-0319.md mode=edit>
✓ ~/docs/meeting-0319.md#decisions — overwritten (4 lines, was 3 lines)
  /undo to revert

 ## Decisions
-│ 1. Ship v2.1 by Friday — no scope changes
-│ 2. Move design review to biweekly
-│ 3. Alice owns stakeholder comms going forward
+│ 1. Ship v2.1 by Friday — no scope changes
+│ 2. Move design review to biweekly
+│ 3. Alice owns stakeholder comms going forward
+│ 4. Adopt new CI pipeline starting next sprint
</𝒞>
```

Block-level addressing is stable — it doesn't shift when other
sections grow or shrink. This is the advantage over line numbers.

### 2.6 Add a new include

The document needs a follow-up section. Add an include to the main
file, then write the content.

**AI →**

```
<𝒞=string:~/docs/meeting-0319.md>
/append

[!include:follow-up]()
</𝒞>
```

**String →**

```
<𝒞=string:~/docs/meeting-0319.md mode=edit>
✓ ~/docs/meeting-0319.md — Appended 2 lines (14 total)

 12  │ [!include:notes]()
 13  │
 14 +│ [!include:follow-up]()
</𝒞>
```

**AI →**

```
<𝒞=string:~/docs/meeting-0319.md#follow-up>
## Follow-up
Next sprint review: 2026-03-26, same time.
Action item review due by Thursday EOD.
</𝒞>
```

**String →**

```
<𝒞=string:~/docs/meeting-0319.md>
✓ created ~/docs/meeting-0319.source/follow-up.md (3 lines)
</𝒞>
```

The document grows by adding new includes — the main file stays
clean and structural.

---

## When to use which approach

| Approach | When |
|----------|------|
| **Plain Markdown** | Short docs, quick notes, most everyday writing |
| **Includes** | Long docs, sections that update independently, content reused across files |

Both approaches use bare content to write and `/replace` for line
edits. The difference is addressing: line numbers for plain docs,
`#block` in the topic for structured docs.

---

## Summary

| Step | Command | What happens |
|------|---------|-------------|
| Write file | bare content (= `/write`) | Create or overwrite a file |
| Write block | bare content to `#block` topic | Write to an include's source file |
| View source | `/edit` | Raw source with line numbers |
| Line edit | `/replace :L5` | Replace a single line |
| Range edit | `/replace :L5-L10` | Replace a line range |
| Undo | `/undo` | Revert last edit (one time only) |
| Append | `/append path` | Add to end of file |
| Verify | `/refresh` | Reload rendered view |
| Snapshot | `/commit path --message "..."` | Named version |
| History | `/log path` | List all versions |
| Time travel | `/open path@version` | Read a past snapshot (read-only) |

The editing flow: **create → edit → verify → commit → repeat.**
