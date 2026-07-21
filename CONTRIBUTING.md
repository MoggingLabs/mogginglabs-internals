# Contributing to MoggingLabs Internals

This repo is an **index/hub**, not a code monorepo. It holds no product code — each tool lives in
its own repository. So "contributing" here means one of two things:

## 1. Adding or updating a tool link

If you've built (or found a home for) a new internal tool:

1. Create the tool in its **own** repository under the
   [MoggingLabs](https://github.com/MoggingLabs) org, with its own `README`, `LICENSE`, and docs.
2. Add a row to the **The tools** table in [README.md](./README.md) linking to that repo.
3. Open a PR here with just that change.

Keep the description to one clear sentence — what the tool does and who it's for.

## 2. Contributing to a specific tool

Head to that tool's own repository and follow **its** `CONTRIBUTING.md`. For example, Highwire's
contribution guide lives at [MoggingLabs/highwire](https://github.com/MoggingLabs/highwire).

## Ground rules (everywhere)

- **Never commit secrets** — no API keys, tokens, `.env` files, or account IDs, in this hub or in
  any tool repo. See [SECURITY.md](./SECURITY.md).
- **Be kind** — see the [Code of Conduct](./CODE_OF_CONDUCT.md).
- Several of our tools automate platforms via internal APIs. Keep their rate-limiting intact and
  don't add functionality meant to mass-target third parties or evade abuse protections for
  harmful ends. Automate your *own* accounts, respectfully.
