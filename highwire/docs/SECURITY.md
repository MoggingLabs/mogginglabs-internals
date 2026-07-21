# Highwire — Security & Token Handling

Highwire holds the keys to your CRM. Treat them accordingly.

## What Highwire stores, and where

| Item | Where it lives | Committed? |
| :--- | :--- | :---: |
| Firebase **refresh token** | local `.env` only | **Never** |
| Short-lived **ID token** | in-memory cache at runtime | **Never** |
| **Location ID** | local `.env` only | **Never** |
| Extracted account data | local `snapshots/` (gitignored) | **Never** |

Only `.env.example` — with **placeholders** — is ever committed.

## Rules

1. **Never commit secrets.** `.env`, `secrets/`, `*.token`, `*.har`, and `snapshots/` are all
   gitignored. Don't override that.
2. **Don't paste tokens into chats, issues, or PRs.** If you need to share a repro, redact.
3. **Scope down when you can.** If an endpoint works with a public private-integration token,
   prefer that over full session auth.
4. **One machine.** Keep the refresh token on your own device. Don't sync `.env` to cloud drives.

## If a token leaks

A refresh token is effectively long-lived access to your CRM. If one is exposed — committed,
pasted, or captured — **assume it's compromised**:

1. **Revoke it at the source immediately.** In GoHighLevel, sign out everywhere / rotate the
   session, and revoke any related private integration key (Settings → Private Integrations).
2. **Purge it from git history** if committed — use `git filter-repo` or BFG, then force-push.
3. **Rotate**, don't just delete. A deleted commit may already be cloned or cached.
4. Open a [security advisory](../../SECURITY.md) if it touched this repo.

## Threat notes

- The Chrome token-grabber extension is intentionally **minimal and auditable** — read every line
  before loading it. It only reads the Firebase auth object from the active GHL tab and copies it
  to your clipboard; it makes no network calls of its own.
- Highwire's client sends requests **only** to GoHighLevel/Firebase endpoints. There is no
  telemetry and no third-party callout.
