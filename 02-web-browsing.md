# Cookbook: Web Browsing

A complete walkthrough of browsing the web through String — opening
URLs, managing tabs, navigating within pages, and combining web
research with document writing.

**Scenario:** Competitive analysis — research multiple company websites,
compare pricing, and compile findings into a report.

→ See [Topics](https://github.com/string-os/string/blob/main/docs/runtime/04-topics.md) for the topic model.
→ See [Navigation](https://github.com/string-os/string/blob/main/docs/runtime/07-navigation.md) for shortcuts and history.

---

## 1. Open a URL (auto-tab)

The AI opens a URL without specifying a topic. String auto-creates
a web tab.

**AI →**

```
<𝒞=string>
/open https://acme-saas.com
</𝒞>
```

**String →**

```
<𝒞=string:web:tab_1 time="2026-03-19T10:00+09:00">
# Acme SaaS — Cloud Platform

Build, deploy, and scale your applications.

## Products
- [Compute][@compute]
- [Storage][@storage]
- [Database][@database]

## Resources
- [Pricing][@pricing]
- [Documentation][@documentation]
- [Blog][@blog]
</𝒞>
```

String assigned `web:tab_1` automatically. The page is converted to
Markdown with auto-generated shortcuts for every link.

---

## 2. Name a tab

For organized research, the AI names its tabs explicitly.

**AI →**

```
<𝒞=string>
/open --tab acme https://acme-saas.com
</𝒞>
```

**String →**

```
<𝒞=string:web:acme time="2026-03-19T10:01+09:00">
# Acme SaaS — Cloud Platform

Build, deploy, and scale your applications.

## Products
- [Compute][@compute]
- [Storage][@storage]
- [Database][@database]

## Resources
- [Pricing][@pricing]
- [Documentation][@documentation]
- [Blog][@blog]
</𝒞>
```

Now the topic is `web:acme` — meaningful and memorable.

---

## 3. Navigate within a tab

The AI follows the pricing link. The tab stays the same, only the
page changes.

**AI →**

```
<𝒞=string:web:acme>
/open @pricing
</𝒞>
```

**String →**

```
<𝒞=string:web:acme time="2026-03-19T10:02+09:00">
# Pricing — Acme SaaS

## Plans

### Starter — $29/mo
- 2 vCPUs, 4GB RAM
- 50GB storage
- Community support

### Pro — $99/mo
- 8 vCPUs, 16GB RAM
- 500GB storage
- Email support, 24h SLA

### Enterprise — Custom
- Dedicated infrastructure
- [Contact Sales][@contact-sales]
- 99.99% SLA

## Compare Plans
- [Full comparison table][@link-1]
- [Calculator][@calculator]

[Back to Home][@back-to-home]
</𝒞>
```

The URL changed within `web:acme` but the session persists. The AI
can `/back` to return to the homepage.

---

## 4. Go back

**AI →**

```
<𝒞=string:web:acme>
/back
</𝒞>
```

**String →**

```
<𝒞=string:web:acme time="2026-03-19T10:03+09:00">
# Acme SaaS — Cloud Platform

Build, deploy, and scale your applications.

## Products
- [Compute][@compute]
- [Storage][@storage]
- [Database][@database]

## Resources
- [Pricing][@pricing]
- [Documentation][@documentation]
- [Blog][@blog]
</𝒞>
```

Back to the homepage. History is per-tab.

---

## 5. Open a second site in a new tab

The AI opens the competitor's site in a separate named tab.

**AI →**

```
<𝒞=string>
/open --tab rival https://rival-cloud.io
</𝒞>
```

**String →**

```
<𝒞=string:web:rival time="2026-03-19T10:04+09:00">
# Rival Cloud — Infrastructure Made Simple

## Services
- [Serverless][@serverless]
- [Containers][@containers]
- [CDN][@cdn]

## Company
- [Pricing][@pricing]
- [About][@about]
- [Changelog][@changelog]
</𝒞>
```

Now the AI has two tabs: `web:acme` and `web:rival`.

---

## 6. Navigate rival's pricing

**AI →**

```
<𝒞=string:web:rival>
/open @pricing
</𝒞>
```

**String →**

```
<𝒞=string:web:rival time="2026-03-19T10:05+09:00">
# Pricing — Rival Cloud

## Plans

### Developer — Free
- 1 vCPU, 1GB RAM
- 10GB storage
- Community forum

### Team — $49/mo
- 4 vCPUs, 8GB RAM
- 200GB storage
- Email support

### Business — $149/mo
- 16 vCPUs, 32GB RAM
- 1TB storage
- Priority support, 4h SLA

[Compare all plans][@compare-all-plans]
[Home][@home]
</𝒞>
```

---

## 7. Fork a tab

The AI wants to explore Rival's comparison page without losing the
pricing view. `/open --tab` forks the session.

**AI →**

```
<𝒞=string:web:rival>
/open --tab rival-compare @compare-all-plans
</𝒞>
```

**String →**

```
<𝒞=string:web:rival-compare time="2026-03-19T10:06+09:00">
# Plan Comparison — Rival Cloud

| Feature | Developer | Team | Business |
|---------|-----------|------|----------|
| vCPUs | 1 | 4 | 16 |
| RAM | 1GB | 8GB | 32GB |
| Storage | 10GB | 200GB | 1TB |
| Support | Forum | Email | Priority |
| SLA | — | — | 4h |
| Price | Free | $49/mo | $149/mo |

[Back to Pricing][@back-to-pricing]
</𝒞>
```

The fork creates `web:rival-compare` — a new independent tab. The
original `web:rival` tab is still on the pricing page, untouched.
Variables and auth are deep-copied into the new session.

---

## 8. Manage tabs

The AI checks what's open.

**AI →**

```
<𝒞=string:web:acme>
/topics web
</𝒞>
```

**String →**

```
<𝒞=string:web:acme>
  web:acme             current: /
  web:rival            current: /pricing
  web:rival-compare    current: /pricing/compare

3 web topics open.
</𝒞>
```

Close the tab that's no longer needed.

**AI →**

```
<𝒞=string:web:acme>
/close web:rival-compare
</𝒞>
```

**String →**

```
<𝒞=string:web:acme>
✓ closed web:rival-compare
</𝒞>
```

---

## 9. Cross-topic workflow: web → file

The AI has gathered enough data. Now it reads from both web tabs and
writes a report to a file topic.

### Read Acme pricing

**AI →**

```
<𝒞=string:web:acme>
/open @pricing
</𝒞>
```

**String →**

```
<𝒞=string:web:acme time="2026-03-19T10:10+09:00">
# Pricing — Acme SaaS

## Plans

### Starter — $29/mo
- 2 vCPUs, 4GB RAM
- 50GB storage
- Community support

### Pro — $99/mo
- 8 vCPUs, 16GB RAM
- 500GB storage
- Email support, 24h SLA

### Enterprise — Custom
- Dedicated infrastructure
- [Contact Sales][@contact-sales]
- 99.99% SLA
</𝒞>
```

### Create the report

The AI reads from both web topics (already in context) and creates
a file — plain Markdown, no special syntax needed.

**AI →**

```
<𝒞=string:~/reports/competitive-analysis.md>
# Competitive Analysis — Cloud Platforms

## Summary
Acme starts at $29/mo (Starter) vs Rival's free tier (Developer).
At mid-tier, Rival ($49 Team) undercuts Acme ($99 Pro) but offers
half the specs. Enterprise pricing requires Acme sales contact;
Rival's Business tier is $149/mo with published specs.

## Price Comparison

| | Acme Starter | Acme Pro | Rival Team | Rival Business |
|---|---|---|---|---|
| Price | $29/mo | $99/mo | $49/mo | $149/mo |
| vCPUs | 2 | 8 | 4 | 16 |
| RAM | 4GB | 16GB | 8GB | 32GB |
| Storage | 50GB | 500GB | 200GB | 1TB |
| Support | Community | Email (24h) | Email | Priority (4h) |

## Recommendation
(pending team discussion)
</𝒞>
```

**String →**

```
<𝒞=string:~/reports/competitive-analysis.md>
✓ created ~/reports/competitive-analysis.md (22 lines)
</𝒞>
```

Web research → file output, all through String topics.

---

## 10. Authenticated sites with $ secrets

When browsing authenticated services, runtime secrets (`$`) handle
credentials. The AI never sees the secret values.

### The document defines the action

```markdown
# In an SFMD app — e.g., ~/apps/github-browser/index.md

`/act.get_repos --org "{org name}"`

​```act.get_repos
GET https://api.github.com/orgs/{org}/repos
  Authorization: Bearer $GITHUB_TOKEN
  org: string (required) "Organization name"
​```

​```act.get_repos.response
{repo_count} = {Response.body.length}
## Repositories ({repo_count})
{Response.body}
​```
```

### The AI invokes it

**AI →**

```
<𝒞=string:app:github-browser>
/act.get_repos --org "acme-corp"
</𝒞>
```

**String →**

```
<𝒞=string:app:github-browser time="2026-03-19T10:20+09:00">
## Repositories (23)
- acme-corp/api — REST API service
- acme-corp/web — Frontend application
- acme-corp/infra — Infrastructure configs
...
</𝒞>
```

The AI used `/act.get_repos` — String injected `$GITHUB_TOKEN` at
runtime. The AI never referenced the token directly and cannot
exfiltrate it.

→ See [State](https://github.com/string-os/string/blob/main/docs/runtime/06-state.md) for the `$` security boundary.

---

## Summary

| Step | Command | What happens |
|------|---------|-------------|
| Open URL | `/open url` | Auto-creates `web:tab_N` |
| Named tab | `/open --tab name url` | Creates `web:name` |
| Navigate | `/open @shortcut` | Move within tab, history grows |
| Go back | `/back` | Previous page in this tab |
| Fork tab | `/open --tab new-name @shortcut` | New tab, copies auth/state |
| List tabs | `/topics web` | All open web topics |
| Close tab | `/close web:name` | Discard tab and its state |
| Cross-topic | Write to `string:~/path` | File topic for output |
| Auth sites | `$SECRET` in action defs | Runtime injects, AI never sees |

The browsing flow: **open → navigate → fork when needed → compile results.**
