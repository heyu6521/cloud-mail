# Cloudflare production deployment

This fork deploys the `mail-worker` directory as the `cloud-mail` Cloudflare Worker. The Vue frontend is built by the Wrangler build hook and uploaded through the Worker static-assets binding; this is not a Pages project.

## Production resources

| Resource | Cloudflare name | Worker binding |
| --- | --- | --- |
| Worker | `cloud-mail` | — |
| D1 | `cloud-mail-db` | `db` |
| KV | `cloud-mail-kv` | `kv` |
| Workers AI | account binding | `ai` |
| Static assets | `mail-worker/dist` | `assets` |
| R2 attachments | `cloud-mail-files` (pending account-level R2 enablement) | `r2` |

Production URL: `https://mail.heyuspace.com`

Runtime variables committed in `mail-worker/wrangler.toml` are limited to non-sensitive settings such as `domain` and `admin`. The `jwt_secret` value is a Cloudflare Worker Secret. Never add JWT secrets, API tokens, passwords, Resend keys, or other credentials to this repository.

## Build and deploy

Cloudflare Workers Builds should use:

- Repository: `heyu6521/cloud-mail`
- Production branch: `main`
- Root directory: `mail-worker`
- Build command: leave empty (Wrangler runs the configured build hook)
- Deploy command: `pnpm run deploy`

For an authenticated command-line deployment, run `pnpm install` and `pnpm run deploy` from `mail-worker`. The configured build hook installs and builds `../mail-vue` before Wrangler uploads the Worker and static assets.

Database initialization and upgrades are implemented by the current source at `/api/init/:secret`. Invoke that endpoint without logging or exposing the secret. A successful run returns `success`; users, including the administrator, set their password through the website registration flow.

## Upgrades, backup, and rollback

Before syncing from `maillab/cloud-mail`, back up D1 with `wrangler d1 export cloud-mail-db --remote --output <backup.sql>` and review upstream changes to `mail-worker/wrangler.toml`, bindings, and initialization code. Preserve the production resource IDs and secret handling when resolving updates.

Cloudflare Worker versions can be inspected with `wrangler versions list --name cloud-mail` and rolled back with `wrangler rollback <VERSION_ID> --name cloud-mail`. D1 data rollback requires restoring a verified export; Worker rollback does not roll back database contents. KV and R2 data should be backed up separately when they contain production data.

Email Routing uses Cloudflare's MX records for `heyuspace.com`. Keep any existing literal-address rules when enabling the catch-all Worker rule. Do not replace MX records if the domain is moved to another mail provider.
