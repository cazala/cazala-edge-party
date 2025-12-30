### `cazala-edge-party` (Cloudflare Worker)

This repo deploys a **route-scoped Cloudflare Worker** that serves the Party playground at:

- **`https://caza.la/party`**

It does this by **reverse-proxying** to the upstream origin:

- **`https://party.caza.la`**

The browser URL stays on **`caza.la/party...`** (no redirects to `*.pages.dev`, `party.caza.la`, or the upstream domain).

### Behavior

- **`GET /party` → 308 → `/party/`**
- **`/party/*` is proxied upstream with `/party` stripped**
  - `/party/` → upstream `/`
  - `/party/assets/x.js` → upstream `/assets/x.js`
  - `/party/some/spa/route` → upstream `/some/spa/route`

The Worker preserves:

- method, query string, request body
- most headers (with safe adjustments)

It adds:

- `x-edge-proxy: cazala-edge-party`

### Configuration

Configured in `wrangler.toml`:

- **Worker name**: `cazala-edge-party`
- **Route**: `caza.la/party*`
- **Upstream**: `vars.UPSTREAM_ORIGIN` (default: `https://party.caza.la`)

### Scripts

- **Typecheck**:

```bash
npm run typecheck
```

- **Deploy (manual)**:

```bash
npm run deploy
```

### GitHub Actions deploy

On push to `main`, `.github/workflows/deploy.yml` will:

- install deps (`npm ci`)
- typecheck (`npm run typecheck`)
- deploy (`wrangler deploy`)

Add these GitHub repo secrets:

- **`CLOUDFLARE_API_TOKEN`**
- **`CLOUDFLARE_ACCOUNT_ID`**

### Acceptance test checklist

Assuming the Worker is deployed and the route is active:

- **`https://caza.la/party`** loads and ends at **`https://caza.la/party/`**
- Hard refresh works on deep SPA routes:
  - open `https://caza.la/party/some/deep/route`
  - refresh → still loads (no 404)
- Static assets load from **`/party/assets/...`** (verify in DevTools Network)
- `https://caza.la/` and other paths continue to work (not served by this Worker)
- No redirects to `party.caza.la`, `party.caza.la`, or any `*.pages.dev` URL



