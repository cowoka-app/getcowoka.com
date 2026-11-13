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

Pages Direct Upload (`wrangler pages deploy`) calls **account-level** APIs
(`/accounts/<id>/pages/projects/...`), not zone/DNS APIs. A token limited to the
**getcowoka.com** zone cannot authorize that route and typically fails with
**Authentication error [code: 10000]** — even if DNS for that domain is in the same account.

**Preferred token type:** **Account API token** (service principal; survives user
leaving). Official Pages docs use this path; Pages is compatible with account-owned
tokens. **User API tokens** also work if you prefer creating them under your profile.

| | Account API token (preferred) | User API token (also OK) |
|--|------------------------------|---------------------------|
| Where | Dashboard → **Manage account** → **Account API tokens** | [My Profile → API Tokens](https://dash.cloudflare.com/profile/api-tokens) |
| Prefix | often `cfat_…` | often `cfut_…` |
| Who can create | Super Admin or member with API Token Provisioning | Any member (limited to their own permissions) |

#### Permissions (only this)

| Scope | Permission (dashboard label) | Access |
|-------|------------------------------|--------|
| **Account** | **Cloudflare Pages** | **Edit** |

That is the same permission Cloudflare’s [Direct Upload CI guide](https://developers.cloudflare.com/pages/how-to/use-direct-upload-with-continuous-integration/) and [Pages REST API](https://developers.cloudflare.com/pages/configuration/api/) ask for (`Pages Write` in API docs).

**Do not add** (and you do **not** need them for this workflow):

| Avoid | Why |
|-------|-----|
| **Account Settings → Edit** | Far too broad (membership / account config). Refuse this. |
| **Account Settings → Read** | Not required here: the Actions workflow already passes `CLOUDFLARE_ACCOUNT_ID`, so Wrangler does not need to list/lookup accounts. Some dashboards only show **Edit** for Account Settings — skip the whole permission. |
| **Account Custom Pages** Read/Edit | Different product (Error Pages), not Cloudflare Pages. |
| Any **Zone** / DNS / Workers Routes permissions | Not used by `pages deploy`. |
| Restricting the token to the **getcowoka.com** domain / one zone | Zone scoping does not limit Pages projects; it often blocks the account Pages API instead. |

#### Resources

1. **Account resources:** include **only** the Cloudflare account that owns the Pages project `getcowoka-com`.
2. **Zone resources:** leave unset / do not add a zone include. You do **not** need “All zones” and you must **not** pin the policy to `getcowoka.com` for deploy.
3. Create token → copy once → put it in the GitHub secret (below).

#### Quick verify (optional)

```bash
curl -sS -o /dev/null -w "%{http_code}\n" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/pages/projects/getcowoka-com"
```

Expect **200**. **401/403** with auth errors → wrong permission name, zone-restricted policy, or wrong account id.

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
