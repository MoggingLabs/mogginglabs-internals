# MoggingLabs Internals — Roadmap

> The plan for the internal tools we're building, and the order we're building them in.
> Nothing here is product code — each tool ships as its own repo and earns a row in the
> [README](./README.md) table when it's real. This file is the map.

## The core idea

**Highwire taught us the pattern:** don't accept a platform's "do it by hand in the UI"
constraint. Wrap its API (public *or* internal) in a typed client, add a human-paced safety
layer, and expose it as something Claude can drive — one tool per action. Highwire does this
for **HighLevel**.

The mistake to avoid is trying to build the big, cross-platform tools first. You can't. A tool
that "sets up a whole client" has to *drive* HighLevel **and** Closebot **and** TeamFollowup
**and** n8n — so each of those platforms needs its own Highwire-style **driver** before anything
can compose them.

So we build **bottom-up, one layer at a time**:

```
Layer 2   Ringmaster                                          ← orchestrator (build LAST)
Layer 1   Spotlight · Watchtower · Ringside · Marquee · Trueup ← compose the drivers
Layer 0   Highwire · Closewire · Followwire · Nodewire         ← one driver per platform (build FIRST)
```

Rule: **you don't build a tool until the layer beneath it exists and works.** Ringmaster is not
started until every Layer-1 piece it needs is proven; a Layer-1 piece is not started until the
Layer-0 drivers it calls are proven.

---

## Layer 0 — Platform drivers (build these first)

Each driver is a single-purpose access layer for **one** platform in our stack. Same shape as
Highwire: typed API/internal-API client → safety/pacing layer → an MCP server (default form
factor, so Layer 1 and Ringmaster can orchestrate it) with one tool per action. Its own repo,
its own auth, no cross-platform logic.

| Driver | Drives | Status | API type | Form factor |
| :--- | :--- | :--- | :--- | :--- |
| [**Highwire**](https://github.com/MoggingLabs/highwire) | HighLevel | ✅ Building | Internal API (replayed) | MCP |
| [**Closewire**](https://github.com/MoggingLabs/closewire) | Closebot | 🚧 Scaffolded | Official API (documented) | MCP |
| [**Followwire**](https://github.com/MoggingLabs/followwire) | TeamFollowup.ai | 🚧 Scaffolded | Official API (**undocumented**) | MCP |
| [**Nodewire**](https://github.com/MoggingLabs/nodewire) | n8n | 🚧 Scaffolded | Official REST API | MCP (adopt `czlonkowski/n8n-mcp`) |

> **Status legend:** ✅ Building = repo + code in progress · 🚧 Scaffolded = repo created with
> README/scaffold + prior-art research, build not yet started. Each repo's git-ignored
> `prompts/RESEARCH.md` holds its full prior-art report (2026-07-22).

### Highwire — HighLevel driver ✅
Built. Drives GoHighLevel via its internal API: contacts, calendars, workflows, sub-accounts,
custom fields. Refresh-token auth, human-paced writes. Reference implementation for every driver
below.
**Prior art (research):** the internal-API auth is a solved problem in ~5 public repos (Firebase +
30-day v2-JWT refresh flows); the **human-paced rate limiter is Highwire's genuine differentiator**
— no prior art exists. Only `thejetmansam/ghl-automation-scraper` is MIT (its anti-ban lessons are
worth porting).

### Closewire — Closebot driver
**Goal:** let Claude configure and read Closebot without touching the UI.
**Why it's a clean first driver:** Closebot exposes a full official API — "everything the UI can
do, the API can do" — so no internal-API replay needed. Lowest-risk driver to build after Highwire.
**Capabilities to expose:** list/create/update bots, edit persona/prompt/objection-handling,
connect a bot to a GHL sub-account, read conversation transcripts, read booking/qualification outcomes.
**Milestones:**
- `v0.1` read-only — list bots, read transcripts + outcomes (feeds Ringside later)
- `v0.2` write — create/update a bot, push persona/prompt
- `v0.3` wiring — connect a bot to a sub-account end-to-end
**Open items:** API key/scopes, rate limits, whether prompts can be fully set via API.
**Prior art (research):** real REST API at `api.closebot.com` (`X-CB-KEY`, ~126 operations) + a
separate `api.closebot.ai/message` runtime endpoint. Persona/prompt maps to editing a typed
drag-and-drop **"Job Flow"**, not one prompt field. MIT `Bleupreneur/closebot-agency-toolkit` ships
the full OpenAPI spec + node schemas — codegen the client from it.

### Followwire — TeamFollowup.ai driver
**Goal:** let Claude manage TeamFollowup's AI **voice** agents + campaigns/cadences and read
call/booking stats.
**Good news (research):** it has an **official, scoped, bearer-token API** (`api.teamfollowup.ai`,
in-app API Keys page) — **no internal replay needed.** It's just **undocumented**, so v0.0 is a
bounded endpoint-mapping spike (get a key, capture the app's live calls), not reverse-engineering.
**Capabilities to expose:** agents (create/update/clone, outcome prompts), campaigns + cadence
actions, contacts (+ kill-switch), calls (history/details → feeds Ringside), dashboard stats
(booking-rate), GHL fields/calendars/tags/pipelines.
**Milestones:**
- `v0.0` endpoint-mapping spike — key + capture live app calls, confirm exact paths
- `v0.1` read stats — dashboard + call history
- `v0.2` write — agents, campaigns, cadence actions
**Note:** actions place **real phone calls** and spend voice-minute credits → pacing + dry-run +
kill-switch are first-class. No open-source prior art — greenfield.

### Nodewire — n8n driver
**Goal:** let Claude manage n8n workflows programmatically instead of clicking in the editor.
**Why:** n8n has a documented REST API and is self-hostable, so this is low-risk. It's the platform
we'll lean on to clone/parameterize per-client automations.
**Capabilities to expose:** list/import/export workflows (workflows-as-JSON), clone a workflow and
set client-specific params, activate/deactivate, read executions (success/failure/stuck).
**Milestones:**
- `v0.1` read — list workflows + executions (feeds Watchtower later)
- `v0.2` write — clone + parameterize a workflow for a new client
- `v0.3` lifecycle — activate/deactivate, bulk ops
**Open items:** self-hosted vs n8n-cloud API differences, credential handling for cloned workflows.
**Prior art (research):** **adopt** the mature MIT MCP [`czlonkowski/n8n-mcp`](https://github.com/czlonkowski/n8n-mcp)
(22k★) for raw API ops — don't rewrite it. Our real build surface is the ~20% no tool does:
**per-client workflow cloning + credential remap** (n8n has no native duplicate, and credential
secrets are write-only) + dedicated activate/deactivate + execution triage.

---

## Layer 1 — Cross-cutting capabilities (build after the drivers)

Each of these **composes the Layer-0 drivers** — it has no direct platform access of its own, it
calls Closewire/Followwire/Nodewire/Highwire. Per the decisions made: **form factor is chosen per tool**
(MCP for interactive, cron/n8n for scheduled monitors); **data store is decided per tool** when we
build it; **findings/alerts surface on a simple internal dashboard** (not push, for now).

| Tool | Does | Composes | Surfaces on |
| :--- | :--- | :--- | :--- |
| **Spotlight** | End-to-end account smoke-test / QA harness | Highwire + Closewire + Followwire | Dashboard (red/green) |
| **Watchtower** | Continuous health monitor across all clients | all drivers | Dashboard |
| **Ringside** | Grade Closebot/TeamFollowup conversations with Claude | Closewire + Followwire | Dashboard (flags) |
| **Marquee** | Metrics warehouse + auto client reports | all drivers | Dashboard + client-facing reports |
| **Trueup** | Snapshot drift detector (sub-account vs canonical) | Highwire | Dashboard (diffs) |

- **Spotlight** — after a client is set up, prove it works: calendar connected? A2P live? test lead
  → Closewire replies → appointment books → Followwire follow-up fires? Bounded; the first Layer-1 tool.
- **Watchtower** — watch KPIs per client (lead volume, booking rate, n8n failures via Nodewire,
  no-show spikes) and flag deviations on the dashboard.
- **Ringside** — sample transcripts (via Closewire/Followwire) and have Claude grade qualification,
  bookings missed, off-brand tone, objection handling; flag bad ones.
- **Marquee** — pull all drivers into one store and generate branded weekly/monthly client reports.
- **Trueup** — read-only diff of a sub-account (via Highwire) vs the canonical snapshot; catch drift.

---

## Layer 2 — Orchestrator (build LAST)

### Ringmaster
The one-command client provisioner. Given a client intake, it drives the whole setup by
**orchestrating the proven pieces** — Highwire (sub-account + snapshot), Closewire (bot), Nodewire
(n8n workflows), Followwire (follow-up), then Spotlight to verify. It is deliberately the last thing
built: it invents no new platform access, it only sequences tools that already work on their own.

---

## Conventions

- **One tool = one repo.** Each ships standalone with its own README/LICENSE, and gets a row added
  to the [README](./README.md) table when it's real.
- **Driver shape:** typed client (official or replayed internal API) → safety/pacing layer → MCP
  server, one tool per action. Copy Highwire.
- **Pacing is mandatory for internal/undocumented APIs.** Any driver that replays internal or
  undocumented endpoints — or uses a captured session token — MUST route every call through a
  **human-paced** layer: jittered human-timed delays, per-hour/day budgets, exponential backoff, a
  circuit breaker, and dry-run. The point is to *humanize* traffic and avoid detection/suspension,
  like Highwire. Sanctioned first-party public APIs still get polite rate-limiting, but the
  humanization bar is highest whenever we're not on an official public API (e.g. Highwire's HighLevel
  internal API; Followwire's undocumented TeamFollowup API + any session-token fallback).
- **Naming:** drivers share a `-wire` family (Highwire, Closewire, Followwire, Nodewire); the
  higher-layer tools keep the circus / tightrope theme (Spotlight, Watchtower, Ringside, Marquee,
  Trueup, Ringmaster).
- **Build order is a dependency order, not a wishlist** — bottom-up, never skip a layer.
