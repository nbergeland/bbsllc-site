# bbsllc.ai

Static site for **BBS LLC · Bergeland Business Solutions** — a boutique data consultancy.

- Single self-contained HTML file (`index.html`), no build step.
- Hosted on **Cloudflare Pages**, deployed from this repo.
- Custom domain: **www.bbsllc.ai** (canonical), with **bbsolutions.biz** 301-redirecting to it.

## Local preview

```bash
# any static server works
python3 -m http.server 8080
# → http://localhost:8080
```

## Deploy

See [`DEPLOY.md`](./DEPLOY.md) for the full Cloudflare Pages + GoDaddy DNS runbook.

## Structure

```
.
├── index.html       # the entire site
├── robots.txt       # search-engine policy
├── README.md        # this file
└── DEPLOY.md        # deployment runbook
```

## Notes

- Fonts loaded from Google Fonts (Fraunces, Instrument Sans, JetBrains Mono).
- No JavaScript dependencies beyond the inline IntersectionObserver reveal.
- All content edits happen in `index.html`. Commit → Cloudflare auto-builds.
