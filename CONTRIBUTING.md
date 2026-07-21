# Contributing to MoggingLabs Internals

Thanks for taking the time to contribute! This repo groups several independent tools, so this
guide covers the shared conventions. Each tool folder may add its own `CONTRIBUTING` notes on
top of these.

## Ground rules

1. **Never commit secrets.** No API keys, tokens, `.env` files, cookies, HAR captures, or
   account IDs — yours or anyone else's. See [SECURITY.md](./SECURITY.md). Every tool ships a
   `.env.example` with placeholders; copy it to `.env` locally and keep `.env` gitignored.
2. **One tool per folder.** Keep changes scoped to the tool you're working on unless you're
   touching shared root docs.
3. **Be kind.** See the [Code of Conduct](./CODE_OF_CONDUCT.md).

## Workflow

```bash
# 1. Fork and clone
git clone git@github.com:<you>/mogginglabs-internals.git
cd mogginglabs-internals

# 2. Branch from main
git checkout -b feat/<tool>-<short-description>

# 3. Make your change inside the relevant tool folder

# 4. Commit with a clear, scoped message
git commit -m "highwire: add opportunity pagination"

# 5. Push and open a PR against main
```

### Commit message format

Prefix with the tool name so history stays readable across the monorepo:

```
highwire: <what changed>
<tool>: <what changed>
```

### Pull requests

- Keep PRs focused — one concern per PR.
- Describe **what** changed and **why**.
- Confirm the "Never commit secrets" checklist in your PR description.
- If you added or changed an internal-API endpoint mapping, update that tool's
  `docs/` reference in the same PR.

## Reporting bugs & requesting features

Open a [GitHub Issue](https://github.com/MoggingLabs/mogginglabs-internals/issues) and label it
with the tool name. Include repro steps, expected vs. actual behavior, and your environment
(OS, Python/Node versions).

## A note on responsible use

Several tools here automate platforms via undocumented/internal APIs. Contributions must keep
the **human-paced rate limiting** intact and must not add functionality whose purpose is to
mass-target third parties or evade abuse protections for harmful ends. Automate your *own*
accounts, respectfully. See each tool's README for the specifics.
