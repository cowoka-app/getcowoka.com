# getcowoka.com

Marketing site for **Cowoka** — coming-soon teaser (privacy & terms drafts stay unpublished until reviewed).

| Domain | Role |
|--------|------|
| **https://getcowoka.com** | Marketing (this repo) |
| **https://cowoka.app** | Product / web app (later; monorepo `cowoka-app/cowoka`) |

## Canonical URLs

- Home (live): https://getcowoka.com/
- Privacy / Terms: **not public yet** — drafts under `drafts/`; `/privacy/` and `/terms/` redirect home until review.

When legal copy is approved, move `drafts/privacy` and `drafts/terms` back to the site root, restore nav links, and redeploy. Then app Account / About can use **https://getcowoka.com/privacy/** (not `cowoka.app/privacy`).

## Stack

Plain static HTML + CSS. No build step. Hosted on **Cloudflare Pages**.

```
/
├── index.html              # Coming soon (published)
├── assets/css/site.css
├── _redirects              # /privacy /terms → /
├── _headers
└── drafts/                 # NOT published — privacy & terms until reviewed
    ├── privacy/index.html
    └── terms/index.html
```

## Local preview

From this repo root:

```bash
# Python
python3 -m http.server 8080

# or Node
npx --yes serve -l 8080
```

Open http://localhost:8080 for the teaser. Draft legal pages are at `/drafts/privacy/` and `/drafts/terms/` locally only (CI excludes `drafts/` from the Cloudflare upload).

## Cloudflare Pages deploy (GitHub Actions + API token)

**Do not** install the **Cloudflare Workers & Pages** GitHub App on this repo. That install asks for org/repo admin + write so Cloudflare can pull source, open deployments, and post checks. We push artifacts instead: GitHub Actions uploads the static tree with a least-privilege Cloudflare API token. Cloudflare never gets GitHub admin.

### Project name vs domain (important)

| What | Value | Where you see it |
|------|-------|------------------|
| **Pages project name** | `getcowoka-com` | Workers & Pages project list; Wrangler `--project-name` |
| **Custom domain** (public URL) | `getcowoka.com` | Pages → Custom domains; browser |

Cloudflare **does not allow dots** in Pages project names (only lowercase letters, digits, hyphens). So the project cannot be named `getcowoka.com`. The workflow uses `getcowoka-com`; you then attach **custom domain** `getcowoka.com` to that project. Seeing `getcowoka.com` in DNS / Custom domains does **not** mean the project is named that.

### 1. Pages “project” / “application” (once)

Cloudflare’s UI often says **Create application** — that is just a **Pages project**, not a separate app product. Name it **`getcowoka-com`** (must match the workflow; dots are invalid).

**Easiest: skip the dashboard.** The workflow runs `wrangler pages project create getcowoka-com` (ignores “already exists”), then `pages deploy`. You do not need a template, Workers starter, or Git connect.

**If you prefer the dashboard** (optional):

1. **Workers & Pages** → **Create** / **Create application**.
2. Avoid framework **templates** and **Connect to Git**.
3. Choose **Get started** → **Drag and drop your files** (Direct Upload), *or* **Pages** → **Use direct upload** if that label appears.
4. Project / application name: **`getcowoka-com`**.
5. You can drag the repo folder once, or cancel/leave empty and let Actions deploy.

### 2. Create a Cloudflare API token (least privilege)

Pages Direct Upload (`wrangler pages deploy`) calls **account-level** APIs
(`/accounts/<id>/pages/projects/...`), not zone/DNS APIs. A token limited to the
**getcowoka.com** zone cannot authorize that route and fails with
**Authentication error [code: 10000]** — that is what broke the failed Actions run.

Cloudflare’s [Direct Upload CI guide](https://developers.cloudflare.com/pages/how-to/use-direct-upload-with-continuous-integration/) creates the token on the **Account API tokens** page. A **User API token** (My Profile → API Tokens → Custom token) with the same permission also works.

| | Account API token (docs path) | User API token (also OK) |
|--|------------------------------|---------------------------|
| Where | Dashboard → **Manage account** → **Account API tokens** | [My Profile → API Tokens](https://dash.cloudflare.com/profile/api-tokens) |
| Create | **Create Token** → **Custom token** → **Get started** | **Create Token** → **Custom token** → **Get started** |
| Who can create | Super Admin or member with API Token Provisioning | Any member (limited to their own permissions) |

#### Permissions (only this)

| Scope | Permission (dashboard label) | Access |
|-------|------------------------------|--------|
| **Account** | **Cloudflare Pages** | **Edit** |

That matches the CI guide and the [Pages REST API](https://developers.cloudflare.com/pages/configuration/api/) (`Pages Write`). No other permission is required when `CLOUDFLARE_ACCOUNT_ID` is already set in GitHub.

**Do not add:**

| Avoid | Why |
|-------|-----|
| **Account Settings → Edit** | Too broad. Refuse this. |
| **Account Settings → Read** | Not needed (account id is in the secret). Some UIs only offer **Edit** — skip Account Settings entirely. |
| **Account Custom Pages** Read/Edit | Error Pages product — not Cloudflare Pages. |
| Any **Zone** / DNS / Workers Routes permissions | Unused by `pages deploy`. |
| Restrict token to zone / domain **getcowoka.com** | Zone scoping blocks the account Pages API → code **10000**. |

#### Resources

1. **Account resources:** include **only** the account that owns the Pages project.
2. **Zone resources:** leave unset / do not add a zone include. Do **not** pin the policy to `getcowoka.com`.
3. Create token → copy once → put it in the GitHub secret (below). Revoke the old zone-scoped token.

#### Quick verify (optional)

```bash
curl -sS -o /dev/null -w "%{http_code}\n" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/pages/projects/getcowoka-com"
```

Expect **200** (project exists) or **404** (token works; create project in step 1). **401/403** / auth errors → wrong permission, zone-restricted policy, or wrong account id.

Find **Account ID**: any zone Overview → API section, or the hex segment in `dash.cloudflare.com/<account_id>/…`.

### 3. GitHub repo secrets

Repo → **Settings** → **Secrets and variables** → **Actions** (repository secrets — not Environments):

| Secret | Value |
|--------|--------|
| `CLOUDFLARE_API_TOKEN` | New token from step 2 |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account ID |

`--project-name=getcowoka-com` is hardcoded in `.github/workflows/deploy-pages.yml` (not a secret).

### 4. Deploy (manual only)

Deploy does **not** run on push. Use **Actions** → **Deploy to Cloudflare Pages** → **Run workflow** when you want a publish. That stages the tree without `drafts/`, ensures project `getcowoka-com` exists, then:

`wrangler pages deploy _publish --project-name=getcowoka-com --branch=main`

### 5. Custom domain + DNS (human)

In Pages project **`getcowoka-com`** → **Custom domains** → add **`getcowoka.com`** (and `www` if desired). At your DNS host (or Cloudflare DNS):

- Apex: CNAME flattening / ALIAS to the Pages target Cloudflare shows, **or** use Cloudflare as nameserver and attach the domain in Pages (recommended).
- `www` → CNAME to `getcowoka.com` or the Pages hostname; optionally redirect www → apex in Pages.

Wait for HTTPS (automatic on Pages). Until DNS is live, the draft is still valid in-repo and on `getcowoka-com.pages.dev`; do not auto-close store/privacy issues until https://getcowoka.com/privacy/ resolves.

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

Privacy and terms live under **`drafts/`** and are **not** uploaded to Pages. They are **draft / not attorney-reviewed**. Grounded in current app concepts (accounts, venues, sessions, location, activity, delete account). Owner must review before publishing or store launch.

## Related

- Product backlog: [Cowoka Product](https://github.com/orgs/cowoka-app/projects/4)
- App privacy wiring: `cowoka-app/cowoka` issue #137 — retarget app URL from `cowoka.app/privacy` to `getcowoka.com/privacy` when this site is the source of truth.
