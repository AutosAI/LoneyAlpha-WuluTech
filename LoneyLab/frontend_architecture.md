# Research: Frontend Architecture

## Tech Stack
- **React 19** + **Vite** + **TypeScript** + **Tailwind CSS** + **shadcn/ui**
- State management via **React Query**

### Path Alias
- `@` → `web/src/` (non-obvious landmine: dual-mode auth `VITE_HOST_MODE`)

### Key Considerations
- **SSE via raw `fetch`** (not the REST client)
- **Zod** at the prefs boundary
- **Dual-mode auth** (`VITE_HOST_MODE`)

## Layout Artifacts

### Files by Purpose

| File | Purpose |
|---|---|
| `src/index.js` | Packaged entry: turns require-time crash into dialog |
| `src/main.js` | Windows, IPC, lifecycle, carries out `policy` decisions |
| `src/policy.js` | Pure: entry URL, where navigation belongs (no store writes) |
| `src/origins.js` | Counts as origin, never by path |
| `src/oauth.js` | System-browser OAuth, intercepted |
| `src/deeplink.js` | `langalpha://` scheme for magic links |
| `src/preload.js` | Renderer bridge: version, platform, capabilities |
| `src/pdf.js` | Renders calling window to PDF (allowlisted options) |
| `src/downloads.js | Gives download visible ending (frameless window) |
| `src/notify.js` | Dock bounce for file landed |
| `src/outage.js` | Replaces blank network-failed window |
| `src/updater.js` | Auto-update, gated on existing feed |
| `src/probe.js` | "Is anything answering" (picker + outage) |
| `src/store.py` | Few scalars in JSON: serverUrl, reachedApp, appChrome, platformChrome, theme |

## Edition System

### Two Editions, One Source

| | `oss` | `saas` |
|---|---|---|
| Entry on first run | server picker | hosted sign-in |
| Platform console | does not exist | shares app window |
| Config | `config/default.json` (committed) | `config/build.json` (gitignored) |
| App name | `LangAlpha OSS` | `LangAlpha` |
| Bundle id | `ai.langalpha.desktop.oss` | `ai.langalpha.desktop` |
| URL scheme | `langalpha-oss://` | `langalpha://` |

### `config/build.json` Role
- **Deployment configuration, not source**
- Origins written from environment at package time
- Overlays gitignored, base YAML stays documented default

### `scripts/build.mjs` Role
- Rewrites `electron-builder.yml` per edition
- Searches output directory for what it just produced
- Ensures two editions don't share feed

## Layout Rules

### Frame/Titlebar
- **Hiding titlebar** hands top strip to page (only safe if page takes it)
- Both SPAs reserve 38px drag region for macOS window buttons
- Feature-detected, never version-detected

### OAuth
- Google refuses OAuth inside embedded webview
- Shell catches authorize navigation, swaps `redirect_to` for loopback
- Port comes from OS via `listen(0)` (RFC 8252 §7.3)

### Scheme Links
- `<scheme>://integrations/login/<name>/callback`
- `<scheme>://integrations/install/<name>/callback`
- Only edition that can finish these advertises them

## Network Failure

### Outage Detection
- `did-fail-load` on main frame (transport failures)
- `did-navigate` with status 500+ (502 from edge = successful error load)
- `net.isOnline()` alone cannot separate offline, captive-portal, unreachable
- Probes `http://cp.cloudflare.com/generate_204` for portal detection

### Controls
- `preload.js` exposes `langalphaOutage` only when `location.protocol === 'file:'`
- Main process independently refuses those channels unless window shows outage page