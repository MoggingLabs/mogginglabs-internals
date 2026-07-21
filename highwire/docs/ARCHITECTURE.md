# Highwire — Architecture

Highwire turns GoHighLevel's internal web-app API into an MCP server that Claude can drive. This
document explains the design, the reverse-engineering method, the safety model, and the build
phases.

## Component map

```
highwire/
├── ghl-extension/        one-click Chrome extension: grabs the Firebase refresh token
├── .env                  local secrets (gitignored): refresh token + location id
├── ghl_client/           the internal-API client (Python)
│   ├── auth.py           refresh-token → short-lived ID token exchange + caching
│   ├── session.py        replays your real header / user-agent fingerprint
│   ├── pacing.py         the safety net (see "Safety model")
│   └── endpoints/        one module per object type
│       ├── workflows.py
│       ├── contacts.py
│       ├── opportunities.py
│       ├── pipelines.py
│       ├── calendars.py
│       └── conversations.py
├── mcp_server/           MCP server exposing every client function as a tool
├── snapshots/            full extracted dumps (gitignored) → feeds dashboards
└── docs/
```

## Auth model — why one capture lasts

GoHighLevel authenticates the web app with **Firebase**:

- A short-lived **ID token** (JWT, ~1 hour) is sent as the bearer on every internal request.
- A long-lived **refresh token** can be exchanged at Firebase's secure-token endpoint for a
  fresh ID token at any time.

Highwire stores only the **refresh token**. `auth.py` exchanges it for an ID token on demand and
caches that token until it nears expiry, then silently refreshes. That's why a single capture
keeps working for a long time — you're persisting the *regenerator*, not the session itself.

## Reverse-engineering method

For each object type we discover the real endpoints by observing the app, never by guessing:

1. Open Chrome DevTools → **Network** tab (filter: Fetch/XHR) while logged into a **test**
   sub-account.
2. Perform the action once in the UI (e.g. open a workflow, create a contact).
3. Capture the request: **URL, method, query params, headers, and JSON payload**.
4. Record the response shape.
5. Document it in [`docs/endpoints.md`](./endpoints.md) and implement a typed client function.

We start **read-only** on a disposable account so discovery itself is low-risk, then map writes.

## Safety model — the pacing layer

Every call routes through `pacing.py`. This is what keeps you on the wire:

| Guard | Behavior |
| :--- | :--- |
| **Human-timed delays** | Randomized think-time between actions (`MIN_DELAY_S`–`MAX_DELAY_S`). Never fixed intervals. |
| **Serial writes** | Writes execute one at a time; reads use a small concurrency pool. |
| **Header fidelity** | Replays the exact header set + user-agent of your real browser session. |
| **Hourly / daily budgets** | Hard ceilings (`MAX_OPS_PER_HOUR`, `MAX_WRITES_PER_HOUR`). Bulk jobs trickle, never burst. |
| **Backoff** | Exponential backoff with jitter on `429` / `403`. |
| **Circuit breaker** | Auto-halt on repeated auth/permission failures so a bad run can't hammer the API. |
| **Dry-run** | `HIGHWIRE_DRY_RUN=1` previews writes without sending them. |

> **Principle:** Highwire should be indistinguishable from an attentive human using the app — and
> never faster than one. Reach comes from coverage, not speed.

## Build phases

Highwire is built and verified one phase at a time; each ends in something testable.

| Phase | Goal |
| :--- | :--- |
| **P0** | Recon: repo skeleton, target the test sub-account, capture the login/token flow. |
| **P1** | Auth: build the token-grabber extension; `auth.py` mints valid ID tokens from a refresh token. |
| **P2** | Reverse-engineer **read** endpoints for all six object types; document them. |
| **P3** | Build the paced **read** client; run one full extraction snapshot. |
| **P4** | Reverse-engineer **write** endpoints — workflow creation first. |
| **P5** | Build the paced **write** client with dry-run + confirmations. |
| **P6** | Wrap everything in the **MCP server**; wire into Claude. |
| **P7** | Prove end-to-end: build a real workflow via MCP; emit a dashboard-ready export. |

See the top-level README roadmap for status.
