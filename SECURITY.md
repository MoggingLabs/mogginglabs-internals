# Security Policy

## Reporting a vulnerability

If you find a security issue in any tool in this repo — especially anything that could **leak
credentials or tokens** — please report it privately first. Open a
[GitHub Security Advisory](https://github.com/MoggingLabs/mogginglabs-internals/security/advisories/new)
or contact the maintainers directly rather than filing a public issue.

We aim to acknowledge reports within a few business days.

> This repo is an **index** and holds no product code. Each tool enforces its own security policy
> in its own repository — but the rules below apply everywhere, here and across every tool repo.

## Secrets never belong in these repos

These are **public** repositories. The following must **never** be committed:

- API keys, private integration tokens (`pit-...`), OAuth secrets
- Firebase auth tokens, refresh tokens, ID tokens, session cookies
- `.env` files, HAR/network captures, browser storage dumps
- Location IDs, account IDs, or any tenant identifier tied to a real account

Every tool provides a `.env.example` with **placeholders only**. Real values live in a local,
gitignored `.env`. If you ever commit a secret by accident:

1. **Revoke/rotate it immediately** at the source (e.g. GoHighLevel → revoke the private
   integration key, sign out to invalidate the session).
2. Remove it from git history (`git filter-repo` or BFG) and force-push.
3. Open a security advisory so we can confirm remediation.

Rotating the credential is the only real fix — deleting the commit is not enough, because it
may already be cloned or cached.
