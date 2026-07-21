<div align="center">

# MoggingLabs Internals

**A monorepo of internal tools we build to run our agency — kept open by default.**

[![License: MIT](https://img.shields.io/badge/License-MIT-6366f1.svg?style=flat-square)](./LICENSE)
[![Made by MoggingLabs](https://img.shields.io/badge/Made%20by-MoggingLabs-0ea5e9?style=flat-square)](https://github.com/MoggingLabs)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-22c55e?style=flat-square)](./CONTRIBUTING.md)

</div>

---

## What this is

`mogginglabs-internals` is where we keep the small, sharp tools we build for ourselves and
decide to share. Each tool lives in its own folder with its own docs, its own setup, and its
own life. This top level exists only to **group them** so they're easy to find and easy to grow.

> We build these to remove busywork from our own agency. If they're useful to you too, take them —
> everything here is open source.

## The tools

| Tool | What it does | Status |
| :--- | :--- | :--- |
| [**Highwire**](./highwire) | Drive GoHighLevel's full internal API from Claude via a custom MCP server — read and write everything the UI can, with built-in human-paced rate limiting. | 🟡 In development |

*More tools will land in their own folders here over time.*

## Repository layout

```
mogginglabs-internals/
├── README.md            ← you are here
├── LICENSE              ← MIT, covers the whole repo
├── CONTRIBUTING.md      ← how to contribute to any tool here
├── CODE_OF_CONDUCT.md
├── SECURITY.md          ← how to report vulnerabilities
└── highwire/            ← first tool: GoHighLevel × Claude
    ├── README.md
    ├── docs/
    ├── assets/
    └── ghl-extension/
```

## License

[MIT](./LICENSE) © MoggingLabs. Use it, fork it, ship it.
