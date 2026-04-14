# Cookbook: Multi-Topic Workflows

Real-world scenarios where the AI works across multiple topics
simultaneously — reading from one, writing to another, running
commands in a third. This is where String's topic model shines.

→ See [Topics](https://github.com/string-os/string/blob/main/docs/runtime/04-topics.md) for the topic model.
→ See [Transport](https://github.com/string-os/string/blob/main/docs/runtime/11-transport.md) for multi-topic messaging.

---

## Scenario 1: API docs → code

Read API documentation from a website, then write a client module
in a local file.

### Open the API docs

**AI →**

```
<𝒞=string>
/open --tab api-docs https://api.acme.com/docs
</𝒞>
```

**String →**

```
<𝒞=string:web:api-docs time="2026-03-19T09:00+09:00">
# Acme API Documentation

## Authentication
All requests require a Bearer token in the Authorization header.

## Endpoints

### GET /users
Returns a list of users.
- `limit` (optional, default: 20)
- `offset` (optional, default: 0)

### POST /users
Creates a new user.
- `name` (required)
- `email` (required)
- `role` (optional, default: "member")

### GET /users/{id}
Returns a single user by ID.

[Authentication Guide][@authentication-guide]
[Rate Limits][@rate-limits]
[Error Codes][@error-codes]
</𝒞>
```

### Read the auth guide for more detail

**AI →**

```
<𝒞=string:web:api-docs>
/open @authentication-guide
</𝒞>
```

**String →**

```
<𝒞=string:web:api-docs time="2026-03-19T09:01+09:00">
# Authentication Guide

## Bearer Token
Include in every request:
```
Authorization: Bearer <your-token>
```

## Token Endpoint
POST https://api.acme.com/auth/token
- `client_id` (required)
- `client_secret` (required)

Returns: `{ "token": "...", "expires_in": 3600 }`

[Back to docs][@back-to-docs]
</𝒞>
```

### Write the client — two topics in one turn

The AI sends commands to two topics simultaneously: going back in
the docs tab and creating the file.

**AI →**

```
<𝒞=string:web:api-docs>
/back
</𝒞>

<𝒞=string:~/src/acme-client.md>
---
title: Acme API Client
---

[!nav:main](./nav/main.md)

# Acme API Client

`/act.auth --client_id "{id}" --client_secret "{secret}"`
`/act.list_users --limit "{number}"`
`/act.create_user --name "{name}" --email "{email}"`
`/act.get_user --id "{user_id}"`

```act.auth
POST https://api.acme.com/auth/token
  client_id: string (required)
  client_secret: string (required)
```

```act.auth.response
{token} = {Response.body.token}
✓ Authenticated (expires in {Response.body.expires_in}s)
```

```act.list_users
GET https://api.acme.com/users
  Authorization: Bearer {token}
  limit: number (optional) "Max results (default: 20)"
  offset: number (optional) "Pagination offset"
```

```act.list_users.response
{user_count} = {Response.body.total}
## Users ({user_count})
{Response.body.users}
```

```act.create_user
POST https://api.acme.com/users
  Authorization: Bearer {token}
  name: string (required) "User's full name"
  email: string (required) "User's email"
  role: string (optional) "member|admin"
```

```act.create_user.response
{new_user_id} = {Response.body.id}
✓ Created user {Response.body.name} (ID: {new_user_id})
```

```act.get_user
GET https://api.acme.com/users/{id}
  Authorization: Bearer {token}
  id: string (required) "User ID"
```

```act.get_user.response
## {Response.body.name}
- **Email:** {Response.body.email}
- **Role:** {Response.body.role}
- **Created:** {Response.body.created_at}
```
</𝒞>
```

**String →**

```
<𝒞=string:web:api-docs time="2026-03-19T09:02+09:00">
# Acme API Documentation
...
</𝒞>

<𝒞=string:~/src/acme-client.md>
✓ created ~/src/acme-client.md (56 lines)
</𝒞>
```

Both topics responded in the same turn. The web tab went back to
the docs index while the file was created.

---

## Scenario 2: Email → report

Check email in an app topic, then update a report in a file topic.

### Check inbox

**AI →**

```
<𝒞=string:app:gmail:work>
/act.inbox --limit "5"
</𝒞>
```

**String →**

```
<𝒞=string:app:gmail:work time="2026-03-19T09:30+09:00">
## Inbox (5 unread)

1. **Q1 Revenue Numbers** — from: cfo@company.com (09:15)
   [Open][@open]
2. **Deploy Approval** — from: devops@company.com (09:10)
   [Open][@open-2]
3. **Design Review Notes** — from: carol@company.com (08:45)
   [Open][@open-3]
4. **Client Feedback** — from: sales@company.com (08:30)
   [Open][@open-4]
5. **Weekly Standup Recap** — from: pm@company.com (08:00)
   [Open][@open-5]
</𝒞>
```

### Read the revenue email

**AI →**

```
<𝒞=string:app:gmail:work>
/open @open
</𝒞>
```

**String →**

```
<𝒞=string:app:gmail:work time="2026-03-19T09:31+09:00">
## Q1 Revenue Numbers

**From:** cfo@company.com
**Date:** 2026-03-19 09:15

Hi team,

Q1 numbers are in:
- Revenue: $2.4M (up 18% QoQ)
- New customers: 47
- Churn: 3.2% (down from 4.1%)
- ARR: $9.6M

Full report attached. Let me know if you need breakdowns.

Best,
CFO

[Reply][@reply] [Forward][@forward] [Back to Inbox][@back-to-inbox]
</𝒞>
```

### Update the report in a file topic

The weekly report uses includes — each section is a separate block.
The AI writes to the `#revenue` block.

**AI →**

```
<𝒞=string:~/reports/weekly.md#revenue>
## Revenue Update (Q1)
- Revenue: $2.4M (+18% QoQ)
- New customers: 47
- Churn: 3.2% (improved from 4.1%)
- ARR: $9.6M
- Source: CFO email, 2026-03-19
</𝒞>
```

**String →**

```
<𝒞=string:~/reports/weekly.md mode=edit>
✓ ~/reports/weekly.md#revenue — overwritten (6 lines, was 1 line)
  /undo to revert

 ## Revenue Update (Q1)
-│ (awaiting Q1 numbers)
+│ - Revenue: $2.4M (+18% QoQ)
+│ - New customers: 47
+│ - Churn: 3.2% (improved from 4.1%)
+│ - ARR: $9.6M
+│ - Source: CFO email, 2026-03-19
</𝒞>
```

Read from email, write to report — two topics, one workflow.

---

## Scenario 3: Build + changelog

Run a build in a bash session, then update the changelog based on
the results.

### Run the build

**AI →**

```
<𝒞=string:bash:build>
cd ~/projects/webapp && npm run build
</𝒞>
```

**String →**

```
<𝒞=string:bash:build time="2026-03-19T10:00+09:00">
> webapp@2.1.0 build
> vite build

✓ 147 modules transformed.
dist/index.html          0.5 kB
dist/assets/app.js       142.3 kB │ gzip: 45.1 kB
dist/assets/vendor.js    89.7 kB  │ gzip: 28.4 kB
dist/assets/style.css    12.1 kB  │ gzip: 3.2 kB

✓ built in 3.2s
</𝒞>
```

### Run tests

**AI →**

```
<𝒞=string:bash:build>
npm test
</𝒞>
```

**String →**

```
<𝒞=string:bash:build time="2026-03-19T10:01+09:00">
> webapp@2.1.0 test
> vitest run

 ✓ src/auth.test.ts (8 tests) 120ms
 ✓ src/api.test.ts (12 tests) 340ms
 ✓ src/ui.test.ts (23 tests) 890ms

 Test Files  3 passed (3)
      Tests  43 passed (43)
   Duration  1.42s
</𝒞>
```

### Update the changelog

The CHANGELOG is a plain Markdown file. The AI uses `/edit` to get
line numbers, then replaces the relevant lines.

**AI →**

```
<𝒞=string:~/projects/webapp/CHANGELOG.md>
/edit
</𝒞>
```

**String →**

```
<𝒞=string:~/projects/webapp/CHANGELOG.md mode=edit>
  1 │ # Changelog
  2 │
  3 │ ## Unreleased
  4 │ (in progress)
  5 │
  6 │ ## [2.0.0] — 2026-02-15
  7 │ - Initial release
</𝒞>
```

**AI →**

```
<𝒞=string:~/projects/webapp/CHANGELOG.md>
/replace :L3-L4
## [2.1.0] — 2026-03-19

### Added
- New authentication flow with OAuth2 support
- API rate limiting middleware

### Changed
- Upgraded Vite to v6.2
- Vendor bundle reduced by 15% (89.7 kB)

### Fixed
- Session timeout handling in auth module

Build: 147 modules, 3.2s. Tests: 43/43 passed.

## Unreleased
</𝒞>
```

**String →**

```
<𝒞=string:~/projects/webapp/CHANGELOG.md mode=edit>
✓ ~/projects/webapp/CHANGELOG.md:L3-L4 — Added 16 lines, removed 2 lines

  2  │
  3 -│ ## Unreleased
  4 -│ (in progress)
  3 +│ ## [2.1.0] — 2026-03-19
  4 +│
  5 +│ ### Added
  6 +│ - New authentication flow with OAuth2 support
  7 +│ - API rate limiting middleware
  8 +│
  9 +│ ### Changed
 10 +│ - Upgraded Vite to v6.2
 11 +│ - Vendor bundle reduced by 15% (89.7 kB)
 12 +│
 13 +│ ### Fixed
 14 +│ - Session timeout handling in auth module
 15 +│
 16 +│ Build: 147 modules, 3.2s. Tests: 43/43 passed.
 17 +│
 18 +│ ## Unreleased
 19  │
</𝒞>
```

### Deploy from the same bash session

**AI →**

```
<𝒞=string:bash:build>
npm run deploy -- --env staging
</𝒞>
```

**String →**

```
<𝒞=string:bash:build time="2026-03-19T10:05+09:00">
> webapp@2.1.0 deploy
> deploy --env staging

Uploading dist/ to staging...
✓ Deployed to https://staging.webapp.example.com
  Version: 2.1.0
  Environment: staging
</𝒞>
```

The bash session remembers the working directory from the first
command — `cd` persisted. Build, test, deploy all in one session.

---

## Common patterns

### Multiple channel tags in one turn

The AI can address any number of topics in a single response:

```
<𝒞=string:web:api-docs>
/open @authentication
</𝒞>

<𝒞=string:~/src/auth.md>
/open #oauth-flow
</𝒞>

<𝒞=string:bash:dev>
npm test -- --filter auth
</𝒞>
```

String processes all three and returns three response blocks. This
is how the AI works in parallel — no serial round trips.

### /info to check topic state

Before acting on a topic, the AI can check what's there:

```
<𝒞=string:app:weather:korea>
/info
</𝒞>
```

Returns: topic type, current URI, history depth, variables, menus,
and available actions — all in one response.

### /topics to see everything open

```
<𝒞=string:~/report.md>
/topics
</𝒞>
```

```
Active topics:

  ~/report.md          file    current: #revenue
  web:api-docs         web     current: /docs/auth
  app:gmail:work       app     current: inbox
  bash:build           bash    cwd: ~/projects/webapp

4 topics open.
```

Filter by type: `/topics web`, `/topics bash`, `/topics file`.

### Topic lifecycle

```
# 1. First message creates the session
<𝒞=string:web:research>
/open https://example.com
</𝒞>

# 2. Session persists across turns
<𝒞=string:web:research>
/open @about
</𝒞>

# 3. Close when done
<𝒞=string:web:research>
/close
</𝒞>
```

No explicit create. No session tokens. The topic exists from first
use to `/close`.

---

## Summary

| Pattern | Topics | Flow |
|---------|---------|------|
| **API docs → code** | `web:` + `file:` | Read docs, write SFMD app |
| **Email → report** | `app:` + `file:` | Read email, update document |
| **Build + changelog** | `bash:` + `file:` | Run commands, write results |
| **Parallel reads** | Any combination | Multiple tags in one turn |

| Command | Multi-topic use |
|---------|-----------------|
| `/topics [type]` | See all open topics |
| `/info` | Check one topic's state |
| `/close [topic]` | Close from anywhere |
| Multiple `<𝒞>` blocks | Address many topics per turn |

The multi-topic flow: **open topics → work in parallel → cross-reference → close when done.**
