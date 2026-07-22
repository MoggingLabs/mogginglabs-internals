<div align="center">

# MoggingLabs Internals

**The index of internal tools we build to run our agency — each one lives in its own repo.**

[![License: MIT](https://img.shields.io/badge/License-MIT-6366f1.svg?style=flat-square)](./LICENSE)
[![Made by MoggingLabs](https://img.shields.io/badge/Made%20by-MoggingLabs-0ea5e9?style=flat-square)](https://github.com/MoggingLabs)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-22c55e?style=flat-square)](./CONTRIBUTING.md)

</div>

---

## What this is

This repo is a **hub** — a single, easy-to-find index of the small, sharp tools we build for
ourselves and decide to share. It intentionally holds **no product code**. Each tool is its own
standalone repository; this page just points you to them.

> We build these to remove busywork from our own agency. If they're useful to you too, take them —
> everything we link here is open source.

## The tools

| Tool | What it does | Repo |
| :--- | :--- | :--- |
| **🎪 Highwire** | Drive GoHighLevel's full internal API from Claude via a custom MCP server — read and write everything the UI can, with built-in human-paced rate limiting. | [MoggingLabs/highwire](https://github.com/MoggingLabs/highwire) |
| **🎙️ Closewire** | Configure Closebot AI appointment-setter bots from Claude via its official API — bots, personas, job-flows, conversations, and booking metrics. *(🚧 scaffold)* | [MoggingLabs/closewire](https://github.com/MoggingLabs/closewire) |
| **🔁 Followwire** | Drive TeamFollowup.ai's AI voice-call follow-up from Claude — agents, campaigns/cadences, contacts, and call stats via its official API. *(🚧 scaffold)* | [MoggingLabs/followwire](https://github.com/MoggingLabs/followwire) |
| **🔌 Nodewire** | Manage n8n workflows from Claude — clone/parameterize per-client automations and triage executions over the REST API. *(🚧 scaffold)* | [MoggingLabs/nodewire](https://github.com/MoggingLabs/nodewire) |

*More tools will be added to this list as we build and open-source them — each in its own repo. The
🚧 scaffolds are new driver repos with READMEs + prior-art research; build not yet started.*

See the **[Roadmap](./ROADMAP.md)** for what's coming and the order we're building it in: one
Highwire-style *driver* per platform first (Closewire for Closebot, Followwire for TeamFollowup,
Nodewire for n8n), then the cross-cutting tools that compose them, then the Ringmaster orchestrator
last.

## Adding a new tool

1. Create the tool in its **own** repository under the [MoggingLabs](https://github.com/MoggingLabs)
   org.
2. Give it its own `README`, `LICENSE`, and docs.
3. Add a row to the table above linking to it.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the details.

## License

The tools are individually licensed in their own repos. This index is [MIT](./LICENSE) © MoggingLabs.
