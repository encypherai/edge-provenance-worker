# Encypher Edge Provenance Worker

Protect every article on your site from the moment it leaves your server.
The Encypher Edge Provenance Worker runs on Cloudflare and embeds invisible
provenance markers into every HTML article before it reaches readers. When
your content is copied, scraped, or fed into AI models, the provenance
travels with it.

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/encypherai/edge-provenance-worker)

No changes to your CMS. No changes to your publishing workflow.

---

## How It Works

**1. Deploy.** Click the button above. Cloudflare clones this worker into
your account. Set a route for your domain, typically `*yourdomain.com/*`.

**2. Connect.** On the first request, the worker registers your domain with
Encypher and caches the signing credentials. No manual setup in the
dashboard is required.

**3. Protect.** Every article response is intercepted at the edge. The
worker extracts visible text, calls the Encypher API to get a signing plan,
and splices provenance markers into the HTML before serving it to the reader.

Markers are invisible. They survive copy-paste, scraping, and aggregation.
When your content appears elsewhere, you can trace it back.

---

## Requirements

An Encypher account is required. Sign up at
[dashboard.encypher.com](https://dashboard.encypher.com). The free plan
covers up to 1,000 signed articles per month.

---

## Supported Platforms

| Platform | Notes |
|---|---|
| WordPress | Classic themes, Block themes |
| Ghost | Casper and custom themes |
| Squarespace | Blog and article pages |
| Webflow | Rich text blocks |
| Substack | Post and newsletter pages |
| Hugo / Jekyll | Standard article layouts |
| News wires | AP, Reuters, and similar formats |
| Custom HTML | Set `ARTICLE_SELECTOR` in config |

Detection is automatic. The worker identifies article content using a
priority-ordered chain and falls back gracefully when no article is found.

---

## Configuration

All configuration is optional. The worker signs articles with zero config
after deployment.

```toml
[vars]
# Override article detection with a CSS selector.
# Supports tag names, .class, and [attr=value] forms.
# Default: auto-detect
ARTICLE_SELECTOR = ".my-article-class"

# Minimum visible text length (characters) to sign.
# Default: 50
MIN_TEXT_LENGTH = "50"

# Maximum visible text length (characters) to sign.
# Default: 51200
MAX_TEXT_LENGTH = "51200"
```

Set your API key as an encrypted secret, not a plaintext var:

```bash
wrangler secret put ENCYPHER_API_KEY
```

The `PROVENANCE_CACHE` KV namespace is provisioned automatically on deploy.
To provision manually, replace the placeholder ID in `wrangler.toml`:

```toml
[[kv_namespaces]]
binding = "PROVENANCE_CACHE"
id = "YOUR_KV_NAMESPACE_ID"
```

---

## Verifying Signed Content

Copy any text from a signed article and paste it at
[encypher.com/verify](https://encypher.com/verify). The verifier extracts
the embedded markers and returns the source domain, signing timestamp, and
document ID.

Check whether a page has been processed via the response header:

```
X-Encypher-Provenance: active
```

Other values: `skipped:oversized`, `skipped:empty`, `skipped:error`.

The worker also exposes `/.well-known/encypher-verify` for domain ownership
verification:

```json
{
  "domain_token": "...",
  "org_id": "...",
  "worker_version": "1.0.0"
}
```

---

## Free vs. Enterprise

The free plan provides full content provenance for up to 1,000 articles
per month. No API key required to start.

Enterprise adds:

- **Leak source identification.** Per-reader marker variation so you can
  identify which copy of an article was leaked and where it went.
- **C2PA compliance.** Each document is bound to a C2PA manifest for
  standards-compliant provenance that works with C2PA-aware tooling.
- **Granular licensing.** Per-sentence rights metadata specifying permitted
  reuse, syndication, and attribution terms.
- **Unlimited volume.**

Set your `ENCYPHER_API_KEY` secret and enterprise features activate
automatically. No worker changes required.

---

## Development

Run the test suite:

```bash
npm test
```

Run a local dev server:

```bash
npm run dev
```

Set your API key in `.dev.vars` (do not commit this file):

```
ENCYPHER_API_KEY=your_key_here
```

Deploy to production:

```bash
npm run deploy
```

---

## License

Encypher Source Available License 1.0. You may use and modify this worker
to deploy on domains you control in connection with the Encypher API. You
may not use it to build a competing service. See LICENSE for full terms.
