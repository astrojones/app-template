# app-template

Minimal starting point for deploying an app to the `astrojones` org via [nuklaut](https://github.com/astrojones/nuklaut).

Push to `main` → image built + pushed to GHCR → deployed to `https://<repo>.astrojones.de` via Traefik on the shared controller.

## Quickstart

1. **Use this template** (GitHub → "Use this template" → name your repo)
2. **Replace** every `__REPO_NAME__` with your actual repo name:
   ```bash
   grep -rl '__REPO_NAME__' . | xargs sed -i 's/__REPO_NAME__/my-app/g'
   ```
3. **Write per-app secrets** to the controller once (skip if your app has no secrets):
   ```bash
   ssh deploy@178.105.203.242 \
     'sudo install -m 600 -o root -g root /dev/stdin /opt/nuklaut/secrets/my-app.env' <<'EOF'
   DATABASE_URL=postgres://...
   MY_API_KEY=sk-...
   EOF
   ```
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
| Per-app | `/opt/nuklaut/secrets/<repo>.env` on controller | SSH once (step 3 above) |
| Org-wide | GitHub org Actions secrets | Already in place; workflow writes them to `_shared.env` on every deploy |

Never put secrets in `.env` files in git or in per-repo GitHub secrets.
