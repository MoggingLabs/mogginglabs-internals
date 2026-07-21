# Highwire Token Grabber (Chrome extension)

A **minimal, auditable** Chrome extension that reads GoHighLevel's Firebase refresh token from
your logged-in session and copies it to your clipboard — so you can paste it into your local
`.env`. It does this **once**; Highwire's client refreshes ID tokens automatically after that.

> 🛠️ **Status:** placeholder — implemented in **Phase 1** of the build (see
> [../docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md#build-phases)).

## What it will (and won't) do

- ✅ Read the Firebase auth object from the **active GoHighLevel tab** only.
- ✅ Copy the refresh token to your clipboard on a button click.
- ❌ Make **no** network requests of its own — no telemetry, no callouts.
- ❌ Store nothing. The token goes to your clipboard and nowhere else.

## Install (once built)

1. Open `chrome://extensions` and enable **Developer mode**.
2. Click **Load unpacked** and select this `ghl-extension/` folder.
3. Log into GoHighLevel, click the Highwire icon → **Grab refresh token**.

Read every line before loading — it's kept small on purpose. Prefer not to install anything? Use
the manual DevTools capture in [../docs/USAGE.md](../docs/USAGE.md#option-b--manual-devtools-capture-no-extension).
