# Deployment runbook — bbsllc.ai

Goal: replace the current GoDaddy Websites + Marketing site with `index.html` from this folder, hosted on **Cloudflare Pages**, with `www.bbsllc.ai` as the canonical domain and `bbsolutions.biz` redirecting to it.

Time estimate: ~25 minutes of clicks plus 0–24h DNS propagation.

---

## 0. Prerequisites

- [ ] GitHub account (any plan; free is fine)
- [ ] Cloudflare account (free; sign up at https://dash.cloudflare.com if you don't have one)
- [ ] Access to GoDaddy account where `bbsolutions.biz` and `bbsllc.ai` are registered
- [ ] `index.html` (and the rest of this folder) ready to push

---

## 1. Push the site to GitHub

Option A — GitHub web UI (easiest, no CLI):

1. https://github.com/new → name it `bbsllc-site` (or anything), Public or Private both fine.
2. Skip the "initialize with README" option — we already have files.
3. After creating, click **uploading an existing file** on the empty-repo page.
4. Drag `index.html`, `README.md`, `DEPLOY.md`, `.gitignore`, `robots.txt` from `~/Documents/Claude/Projects/BBS/` into the upload area.
5. Commit message: `initial site`.

Option B — `gh` CLI from this folder:

```bash
cd ~/Documents/Claude/Projects/BBS
git init
git add .
git commit -m "initial site"
gh repo create bbsllc-site --public --source=. --push
```

✅ Done when: the repo on github.com shows `index.html`, `README.md`, `.gitignore`, `robots.txt`, `DEPLOY.md`.

---

## 2. Connect Cloudflare Pages

1. https://dash.cloudflare.com → **Workers & Pages** → **Create** → **Pages** tab → **Connect to Git**.
2. Authorize Cloudflare to access GitHub (one-time). Pick the `bbsllc-site` repo.
3. Build settings:
   - Production branch: `main`
   - Framework preset: **None**
   - Build command: *(leave blank)*
   - Build output directory: `/`
4. **Save and Deploy**. First deploy takes ~30 seconds.

✅ Done when: you see a green "Success" badge and a URL like `https://bbsllc-site.pages.dev`. Open it. Confirm the new BBS site renders.

---

## 3. Add custom domains in Cloudflare Pages

In the Pages project → **Custom domains** → **Set up a custom domain** for each of:

- `bbsllc.ai`
- `www.bbsllc.ai`
- `bbsolutions.biz`
- `www.bbsolutions.biz`

For each, Cloudflare will tell you whether the domain's nameservers are already on Cloudflare (then it auto-wires) or not (then it gives you a CNAME target like `bbsllc-site.pages.dev`).

**Recommendation:** move both domains' DNS to Cloudflare (free) for cleaner management. To do that:

1. In Cloudflare → **Add a Site** → enter `bbsllc.ai` → choose Free plan.
2. Cloudflare scans existing GoDaddy records and copies them. **Important:** confirm your MX records (email at `bbsllc.ai`) are present in Cloudflare's copied set before continuing — if any are missing, add them manually now.
3. Cloudflare gives you two new nameservers (e.g. `ada.ns.cloudflare.com`, `kirk.ns.cloudflare.com`).
4. In GoDaddy → `bbsllc.ai` domain → **Nameservers** → change to **Custom** → paste the two Cloudflare NS records → save.
5. Repeat for `bbsolutions.biz`.
6. Wait for Cloudflare to confirm propagation (usually 5–60 min, can be up to 24h).

If you'd rather keep DNS at GoDaddy, skip the nameserver change and use the CNAME-only approach below.

---

## 4. Point DNS at the Pages project

### Path A — DNS at Cloudflare (recommended)

In Cloudflare → DNS, for **bbsllc.ai**:

| Type  | Name | Target               | Proxy |
|-------|------|----------------------|-------|
| CNAME | @    | bbsllc-site.pages.dev | Proxied (orange cloud) |
| CNAME | www  | bbsllc-site.pages.dev | Proxied (orange cloud) |

For **bbsolutions.biz** (the redirect-source domain):

| Type  | Name | Target               | Proxy |
|-------|------|----------------------|-------|
| CNAME | @    | bbsllc-site.pages.dev | Proxied |
| CNAME | www  | bbsllc-site.pages.dev | Proxied |

Then create the redirect (so visiting `bbsolutions.biz` lands on `bbsllc.ai`):

1. Cloudflare → **Rules** → **Redirect Rules** → **Create rule** at the `bbsolutions.biz` zone.
2. Name: `Redirect bbsolutions.biz to bbsllc.ai`.
3. Match: *Hostname equals* `bbsolutions.biz` OR `www.bbsolutions.biz` (use the `or` operator).
4. Then: *Static* redirect to `https://www.bbsllc.ai${http.request.uri}` with status 301 and "Preserve query string" enabled.
5. Deploy.

### Path B — Keep DNS at GoDaddy

In GoDaddy DNS for **bbsllc.ai**:
- Delete any existing A/AAAA records for `@` and `www`.
- Add CNAME `www` → `bbsllc-site.pages.dev`.
- For the apex (`@`), GoDaddy doesn't support CNAME flattening, so use Cloudflare's published Pages IPs as A records, or move just this domain's DNS to Cloudflare. Cloudflare's docs list the current Pages IPs.
- Cancel the GoDaddy Websites + Marketing subscription once the new site is verified live.

In GoDaddy DNS for **bbsolutions.biz**: same pattern, plus you'll need to set up the redirect through GoDaddy's Domain Forwarding feature (less robust than Cloudflare's redirect rule).

---

## 5. Cancel the GoDaddy Websites + Marketing plan

Only after `https://www.bbsllc.ai/` resolves to the new site and SSL is green:

- GoDaddy → My Products → Websites + Marketing → cancel the subscription.
- Domains stay registered separately; only the website-builder product gets cancelled.

---

## 6. Post-launch checklist

- [ ] `https://www.bbsllc.ai/` loads the new site over HTTPS
- [ ] `https://bbsllc.ai/` redirects to `https://www.bbsllc.ai/`
- [ ] `https://bbsolutions.biz/` and `https://www.bbsolutions.biz/` 301 → `https://www.bbsllc.ai/`
- [ ] `mailto:contact@bbsllc.ai` still works (test send-and-receive)
- [ ] Mobile rendering looks right (open on phone)
- [ ] Calendly link works
- [ ] LinkedIn links work
- [ ] Google Search Console: add `bbsllc.ai` as a property and submit `https://www.bbsllc.ai/sitemap.xml` (you'll want to add `sitemap.xml` later — `robots.txt` references it)

---

## What I (Claude/Cowork) can do for you

✅ File ops (already done): write `index.html`, `README.md`, `.gitignore`, `robots.txt`, this runbook.
✅ Drive GitHub web UI: navigate, click "New repo", upload files (you sign in / handle MFA).
✅ Drive Cloudflare Pages flow: connect repo, set build settings, deploy, add custom domain.
⚠️ I'll **surface** the exact GoDaddy DNS records to change but won't click "Save" — you approve the actual writes.
🚫 I won't accept new ToS, create accounts, or cancel subscriptions on your behalf.

If you want me to drive any of the above in this session, install / enable the **Claude in Chrome** extension and tell me — I'll take it from there.
