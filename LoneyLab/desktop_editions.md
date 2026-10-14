# Research: Desktop Edition Architecture

## Two Editions, One Source

### OSS vs SaaS Edition

| Feature | `oss` | `saas` |
|---|---|---|
| Entry on first run | server picker | hosted sign-in |
| Platform console | does not exist | shares app window |
| Config | `config/default.json` (committed) | `config/build.json` (gitignored) |
| App name | `LangAlpha OSS` | `LangAlpha` |
| Bundle id | `ai.langalpha.desktop.oss` | `ai.langalpha.desktop` |
| URL scheme | `langalpha-oss://` | `langalpha://` |

### `src/config.js` Role
- Merges the two editions
- Refuses to start `saas` build without `platformOrigin`
- `app.setName(config.appName)` in `main.js`

### `scripts/build.mjs` Role
- Rewrites `electron-builder.yml` per edition
- Searches output directory for what it just produced
- Asserts both tables equal in tests

### Build Commands
```bash
# OSS (default, no env vars)
pnpm run dist

# SaaS (requires env vars)
DESKTOP_EDIT=saas \
DESKTOP_APP_ORIGIN="https://..." \
DESKTOP_PLATFORM_ORIGIN="https://..." \
pnpm run dist
```

## Key Constraints
- Cannot build both editions simultaneously (race on `config/build.json`)
- Separate outputs: `dist/${EDITION}`
- Update manifests named for channel: `latest-mac.yml`, `latest.yml`
- Signing and notarization wired on CI for both editions

## OAuth & Deep Links
- Generic OAuth catcher without SPA involvement
- `setWindowOpenHandler` returns `deny`
- `begin()` refuses flows whose `redirect_to` not in origins
- `langalpha://` scheme carries 3 kinds of links