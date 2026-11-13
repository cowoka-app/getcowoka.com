# getcowoka.com

Marketing site for **Cowoka** — coming soon, privacy, and terms.

| Domain | Role |
|--------|------|
| **https://getcowoka.com** | Marketing (this repo) |
| **https://cowoka.app** | Product / web app (later; monorepo `cowoka-app/cowoka`) |

## Canonical URLs

- Home: https://getcowoka.com/
- Privacy: https://getcowoka.com/privacy/
- Terms: https://getcowoka.com/terms/

App Account / About links should open **https://getcowoka.com/privacy/** (not `cowoka.app/privacy`).

## Stack

Plain static HTML + CSS. No build step. Hosted on **Cloudflare Pages**.

```
/
├── index.html           # Coming soon
├── privacy/index.html
├── terms/index.html
├── assets/css/site.css
├── _redirects           # trailing-slash aliases
└── _headers
```

## Local preview

From this repo root:

```bash
# Python
python3 -m http.server 8080

# or Node
npx --yes serve -l 8080
```

Open http://localhost:8080 — paths `/privacy/` and `/terms/` work via folder `index.html`.

## Cloudflare Pages setup

1. Cloudflare Dashboard → **Workers & Pages** → **Create** → **Pages** → Connect to Git.
2. Select org repo `cowoka-app/getcowoka.com`.
3. Build settings:
   - **Framework preset:** None
   - **Build command:** *(leave empty)*
   - **Build output directory:** `/` (repo root)
4. Save and deploy. Note the `*.pages.dev` preview URL.
5. **Custom domains** → add `getcowoka.com` (and `www` if desired).
6. At your DNS host (or Cloudflare DNS if the domain is on CF):
   - Apex: CNAME flattening / ALIAS to the Pages target Cloudflare shows, **or** use Cloudflare as nameserver and add the domain in Pages (recommended).
   - `www` → CNAME to `getcowoka.com` or the Pages hostname; optionally redirect www → apex in Pages.
7. Wait for HTTPS certificate (automatic on Pages).

Until DNS is live, the draft is still valid in-repo and on `*.pages.dev`; do not auto-close store/privacy issues until https://getcowoka.com/privacy/ resolves.

## Waitlist (not implemented)

Hero includes a hidden `[data-waitlist-slot]` region and HTML comments marking where a single email field should go.

**Recommended path (stays on Cloudflare):**

1. Unhide `.waitlist-slot` and add `<form id="waitlist">` with `email` + submit.
2. Add a Pages Function or Worker at `POST /api/waitlist` that:
   - validates email
   - writes to **KV** (`WAITLIST` binding) or forwards to an email tool
3. Optionally sync to **Buttondown** via their API from the Worker for newsletters.

**Simpler alternative:** point the form `action` at **Formspree** (or Buttondown embed) with no Worker — fine for early beta volume; migrate to Worker + KV when you want ownership of the list.

Do not collect emails until spam protection (Turnstile) and a privacy blurb on the form are ready.

## Legal copy

Privacy and terms are **draft / not attorney-reviewed**. Grounded in current app concepts (accounts, venues, sessions, location, activity, delete account). Owner must review before store launch.

## Related

- Product backlog: [Cowoka Product](https://github.com/orgs/cowoka-app/projects/4)
- App privacy wiring: `cowoka-app/cowoka` issue #137 — retarget app URL from `cowoka.app/privacy` to `getcowoka.com/privacy` when this site is the source of truth.
