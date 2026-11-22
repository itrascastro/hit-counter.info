# Cloudflare Setup

hit-counter.info runs as a static site (GitHub Pages, `gh-pages` branch) behind Cloudflare, with a
Cloudflare Worker + Durable Objects that serve the counters.

## What this repo deploys

- Public static website (GitHub Pages, custom domain via `CNAME`).
- Worker in `cloudflare-worker.js`.
- Durable Object `CounterDurableObject`: one counter per ID.
- Endpoints:
  - `/c/:id.svg`
  - `/preview.svg`
  - `/api/counters/:id`
  - `/api/styles`
  - `/health`

## Requirements

1. The domain must go through Cloudflare (nameservers pointing to Cloudflare).
2. Wrangler installed and authenticated (`npx wrangler login`, or `CLOUDFLARE_API_TOKEN`).

## Deploy

```bash
npx wrangler deploy
```

The first deploy creates the SQLite Durable Object namespace defined in `wrangler.jsonc`.

## Worker routes

Only the counter service is intercepted; the rest of the domain stays a static site:

- `hit-counter.info/c/*`
- `hit-counter.info/api/*`
- `hit-counter.info/preview.svg*`
- `hit-counter.info/health*`
- the same four on `www.hit-counter.info`

## Abuse protection

Each counter accepts up to 1,000 hits per UTC day (2% of a 50,000 requests/day budget). Beyond that
the counter is blocked until the next UTC midnight, answering `204` with no body.

## Available styles

`split`, `score`, `terminal`, `neon`, `lcd`, `nixie`, `odometer`, `dots`, `badge`, `pixel`, `chalk`.

## Examples

Real counter image:

```html
<img src="https://hit-counter.info/c/my-site.svg?style=split&digits=6&theme=dark" alt="hit counter" style="width:100%;height:auto">
```

Preview without incrementing:

```text
https://hit-counter.info/preview.svg?style=score&digits=6&theme=dark&value=90812
```

Custom colors:

```text
https://hit-counter.info/preview.svg?style=terminal&digits=6&theme=dark&value=31337&accent=00ff88&bg=0a0a0a&color=00ff88&radius=16
```

JSON status:

```text
https://hit-counter.info/api/counters/my-site
```
