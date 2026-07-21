# Highwire — Usage Guide

A step-by-step guide from zero to talking to your GoHighLevel account through Claude.

> ⚠️ Read the [responsible-use disclaimer](../README.md#️-responsible-use-disclaimer) first.
> Highwire uses GoHighLevel's internal API — likely against their ToS. **Use a test/throwaway
> sub-account until you're confident.**

---

## 1. Install prerequisites

- [Python 3.11+](https://www.python.org/downloads/)
- [Node.js 18+](https://nodejs.org/)
- [Claude Code](https://claude.com/claude-code) (or Claude Desktop / another MCP client)
- Google Chrome (recommended for token capture)

## 2. Get the code

```bash
git clone git@github.com:MoggingLabs/mogginglabs-internals.git
cd mogginglabs-internals/highwire
```

## 3. Find your Location ID

Log into GoHighLevel and open the sub-account you want to automate. The Location ID is in the URL:

```
app.yourdomain.com/location/⟨THIS-IS-YOUR-LOCATION-ID⟩/dashboard
```

Copy it — you'll paste it into `.env` in the next step.

## 4. Capture your refresh token

You have two options. **Both put the token only on your machine.**

### Option A — the Highwire Chrome extension (easiest)

1. Open `chrome://extensions`, enable **Developer mode**.
2. Click **Load unpacked** and select the [`ghl-extension/`](../ghl-extension) folder.
3. Log into GoHighLevel, click the Highwire extension icon → **Grab refresh token**.
4. It copies the token to your clipboard.

### Option B — manual DevTools capture (no extension)

1. Log into GoHighLevel in Chrome.
2. Open **DevTools → Application → Storage** (Local Storage / IndexedDB) and locate the Firebase
   auth entry, **or** DevTools → **Network**, trigger any action, and read the refresh token from
   the auth request. (Exact location is documented in [ARCHITECTURE.md](./ARCHITECTURE.md).)
3. Copy the refresh token value.

## 5. Configure `.env`

```bash
cp .env.example .env
```

Open `.env` and paste your values:

```dotenv
GHL_REFRESH_TOKEN=1//your-real-refresh-token
GHL_LOCATION_ID=your-location-id
```

> 🔒 `.env` is gitignored. **Never** paste real tokens anywhere else, and never commit them.

## 6. Install dependencies

```bash
python -m venv .venv
. .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install -e .
npm install
```

## 7. Register the MCP server with Claude

```bash
# Claude Code
claude mcp add highwire -- python -m highwire.mcp_server
```

Restart Claude Code, then confirm Highwire's tools are listed with `/mcp`.

## 8. Use it

Talk to Claude naturally:

| You say | Highwire does |
| :--- | :--- |
| *"Pull the course sales sequence and map every step."* | Finds the workflow by fuzzy name, extracts every node. |
| *"List my pipelines and how many opportunities are in each stage."* | Reads pipelines + opportunities, summarizes. |
| *"Build an abandoned-cart workflow with a 3-email sequence."* | Drafts triggers/conditions/actions, shows a dry-run, then creates it on confirmation. |
| *"Export everything to a snapshot for my dashboard."* | Runs a paced full extraction into `snapshots/`. |

### Safety while you work

- Start with `HIGHWIRE_DRY_RUN=1` in `.env` so writes are previewed, not sent.
- Keep the default pacing values. Lower them if you want to be *more* cautious; raising them
  raises suspension risk.
- If you see repeated `403`/`429`, **stop** — the circuit breaker will halt automatically. Wait,
  then resume.

## Troubleshooting

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| `auth failed` on start | Refresh token expired/revoked | Re-capture the token (step 4). |
| `403` on a specific object | Missing scope/permission on that sub-account | Confirm your GHL user can do it in the UI. |
| Frequent `429` | Pacing too aggressive | Lower `MAX_OPS_PER_HOUR`, raise `MIN_DELAY_S`. |
| Claude doesn't see the tools | MCP not registered | Re-run step 7, restart Claude, check `/mcp`. |

---

Need the design details? See [ARCHITECTURE.md](./ARCHITECTURE.md). Security specifics live in
[SECURITY.md](./SECURITY.md).
