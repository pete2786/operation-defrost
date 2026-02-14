# Operation Defrost

Civic engagement SPA deployed to GitHub Pages at **operationdefrost.app**.

## Architecture

- **No build tools.** Pure vanilla HTML/CSS/JS with Tailwind CDN.
- Single `index.html` is the entire app shell (hash-based SPA router).
- Issue data lives in JSON files under `data/`.
- Deployed via GitHub Pages from `main` branch. Push to main = deploy.

## Key Files

- `index.html` — App shell: router, issue selector, content block toggles, tone selector, rep lookup, template composition engine, social sharing
- `data/issues.json` — Issue index (id, title, summary, image, date, status)
- `data/issues/{id}.json` — Full issue data (content blocks, tones, meta)
- `issue/{id}/index.html` — **Social sharing stubs** (see below)
- `CNAME` — Custom domain config

## Adding a New Issue

1. **Create the issue JSON** at `data/issues/{id}.json` with:
   - `id`, `title`, `subtitle`, `urgentBanner` (or null)
   - `contentBlocks[]` — each with `id`, `label`, `category`, `defaultSelected`, `contextText`, `templateFragment`
   - `tones[]` — each with `id`, `label`, `description`, `color`, `subject`, `intro` (must contain `{selected_blocks}`), `asks[]`, `closing`
   - `meta` — `ogTitle`, `ogDescription`, `ogImage`, `shareText`

2. **Add to the issue index** in `data/issues.json` with `id`, `title`, `summary`, `image`, `date`, `status: "active"`

3. **Create a social sharing stub** at `issue/{id}/index.html` — This is critical for correct Facebook/Twitter previews. Social crawlers don't execute JS, so they need a static HTML file with the correct OG meta tags baked in. The stub redirects browsers to the SPA hash route. Copy an existing stub and update the meta values and redirect URL.

4. **Push to main** — GitHub Pages deploys automatically.

## Social Sharing Stubs (Important)

Social media crawlers (Facebook, Twitter) don't execute JavaScript. Since this is a hash-routed SPA, crawlers always see the static meta tags from `index.html` regardless of which issue is being shared.

**Solution:** Each issue has a lightweight HTML stub at `issue/{id}/index.html` that:
- Contains the correct OG/Twitter meta tags for that specific issue
- Redirects browsers to the SPA via `window.location.replace('https://operationdefrost.app/#/issue/{id}')`
- Shows a fallback link for noscript users

Share URLs point to `/issue/{id}/` (the stub path), not `/#/issue/{id}` (the hash route).

After updating meta tags, use the [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/) to flush cached previews.

## Rep Lookup

Uses whoismyrepresentative.com API via CORS proxy (codetabs primary, corsproxy.io fallback). The proxy list is in `index.html` in the `lookupRep()` function. Proxies can go down — if rep lookup breaks, test alternatives and update the proxy list.

## Images

All issue images are from Wikimedia Commons (free, no attribution required for most). Card thumbnails use 600px width, OG images use 1200px width via Wikimedia's thumb URL pattern.

## Caching

JSON fetches use `{ cache: 'no-store' }` to prevent stale data after updates.
