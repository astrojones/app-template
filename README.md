# app-template

Minimal starting point for deploying an app to the `astrojones` org via [nuklaut](https://github.com/astrojones/nuklaut).

Push to `main` → image built + pushed to GHCR → deployed to `https://<repo>.astrojones.de` via Traefik on the shared controller. No SSH access required.

## Quickstart

1. **Use this template** (GitHub → "Use this template" → name your repo)
2. **Replace** every `__REPO_NAME__` with your actual repo name:
   ```bash
   grep -rl '__REPO_NAME__' . | xargs sed -i 's/__REPO_NAME__/my-app/g'
   ```
3. **Add per-app secrets** as a single GitHub repo secret named `APP_ENV` (skip if your app has none):

   Repo → Settings → Secrets and variables → Actions → New repository secret:
   - **Name:** `APP_ENV`
   - **Value:** multiline env file, e.g.:
     ```
     DATABASE_URL=postgres://user:pass@host/db
     MY_API_KEY=sk-...
     ```

   The deploy workflow writes this to the controller on every push — no SSH needed.

4. **Replace the Dockerfile** with one that actually builds your app.
5. **Push to `main`** — first deploy runs automatically.

## Files

| File | Purpose |
|------|---------|
| `.github/workflows/deploy.yml` | Calls org reusable; no changes needed |
| `.nuklaut/deployment.yml` | Ingress config, optional DB, auth gate |
| `docker-compose.yml` | Service definition; no ports/labels |
| `Dockerfile` | Replace with your own |

## Customising `deployment.yml`

The file has commented examples for the common cases:

- **Path split** (`/api` → backend, `/` → frontend on same host)
- **Postgres + Redis** (`spec.databases` — nuk provisions creds, injects `DATABASE_URL`/`REDIS_URL`)
- **OAuth gate** (`auth: oauth` — GitHub login via org oauth2-proxy)
- **Custom compose file** (`spec.compose.file`)

See [nuklaut/docs/deployments.md](https://github.com/astrojones/nuklaut/blob/main/docs/deployments.md) for the full manifest reference.

## Image naming

The reusable workflow publishes to:

```
ghcr.io/astrojones/<repo>/<repo>:{sha,latest}
```

`docker-compose.yml` references `:latest`; nuk always pre-pulls the latest before `compose up`.

## Secrets

| Type | Where | How |
|------|-------|-----|
| Per-app | `APP_ENV` GitHub repo secret | Repo → Settings → Secrets → `APP_ENV` (multiline key=value) |
| Org-wide | GitHub org Actions secrets | Already in place; managed by the org admin |

Never put secrets in `.env` files committed to git.
