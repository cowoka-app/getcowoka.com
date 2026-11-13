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

## Cloudflare Pages deploy (GitHub Actions + API token)

**Do not** install the **Cloudflare Workers & Pages** GitHub App on this repo. That install asks for org/repo admin + write so Cloudflare can pull source, open deployments, and post checks. We push artifacts instead: GitHub Actions uploads the static tree with a least-privilege Cloudflare API token. Cloudflare never gets GitHub admin.

### 1. Create an empty Pages project (once)

Direct Upload needs a project name before the first CI deploy:

1. Cloudflare Dashboard → **Workers & Pages** → **Create** → **Pages**.
2. Choose **Upload assets** / Direct Upload (not “Connect to Git”).
3. Project name: **`getcowoka-com`** (must match the workflow).
4. You can skip uploading files in the UI; the first Actions run will deploy.

### 2. Create a Cloudflare API token (least privilege)

1. [API Tokens](https://dash.cloudflare.com/profile/api-tokens) → **Create Token** → **Custom token**.
2. Permissions (exact dashboard labels):

   | Scope | Permission | Access |
   |-------|------------|--------|
   | **Account** | **Cloudflare Pages** | **Edit** |
   | **Account** | **Account Settings** | **Read** *(recommended)* |

   Official Pages CI docs require **Cloudflare Pages → Edit**. **Account Settings → Read** is optional for the deploy API itself when `accountId` is passed as a secret, but Wrangler sometimes needs it for account lookup — include it unless you verify deploy works without it.
3. **Account Resources**: include only the Cloudflare account that owns the Pages project.
4. Create token → copy once.

Find **Account ID**: any zone Overview → API section, or the hex segment in `dash.cloudflare.com/<account_id>/…`.

### 3. GitHub repo secrets

Repo → **Settings** → **Secrets and variables** → **Actions**:

| Secret | Value |
|--------|--------|
| `CLOUDFLARE_API_TOKEN` | Token from step 2 |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account ID |

Project name is hardcoded in `.github/workflows/deploy-pages.yml` as `getcowoka-com` (not a secret).

### 4. Deploy

Push to `main` (or run **Actions** → **Deploy to Cloudflare Pages** → **Run workflow**). No build step — the workflow deploys the repo root with `wrangler pages deploy . --project-name=getcowoka-com`.

### 5. Custom domain + DNS (human)

In the Pages project → **Custom domains** → add `getcowoka.com` (and `www` if desired). At your DNS host (or Cloudflare DNS):

- Apex: CNAME flattening / ALIAS to the Pages target Cloudflare shows, **or** use Cloudflare as nameserver and attach the domain in Pages (recommended).
- `www` → CNAME to `getcowoka.com` or the Pages hostname; optionally redirect www → apex in Pages.

Wait for HTTPS (automatic on Pages). Until DNS is live, the draft is still valid in-repo and on `*.pages.dev`; do not auto-close store/privacy issues until https://getcowoka.com/privacy/ resolves.

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
