# Umami deployment runbook

## Production architecture

- Source fork: `caboosesw/umami` (upstream: `umami-software/umami`)
- Permanent WSL checkout: `/home/barry/src/umami`
- Hosting: Vercel project `umami` in team `caboosesws-projects`
- Production URL: <https://umami-beige-eta-20.vercel.app>
- Database: Vercel-managed Neon project `umami-db` (`blue-brook-44905862`), branch `main`
- Neon organization: `Vercel: caboosesw's projects` (`org-wild-hall-30610640`)

Vercel is configured with the Next.js preset, Node.js 24.x, and the build command:

```sh
git init -q && npx --yes pnpm@12.3.4 run build
```

The temporary `git init` is required for CLI deployments because Vercel CLI omits `.git` and `.gitignore`, while Umami's Biome configuration reads Git ignore metadata during the build. It affects only Vercel's disposable build directory.

## Secret and environment handling

Vercel stores the production, preview, and development values. Never commit or paste their values into issues, pull requests, logs, or chat.

Required Umami variables:

- `DATABASE_URL` (pooled Neon connection; provided by the integration)
- `DIRECT_DATABASE_URL` (unpooled Neon connection for migrations)
- `APP_SECRET`
- `TWO_FACTOR_ENCRYPTION_KEY`

The Vercel-Neon integration also manages its standard `DATABASE_URL_UNPOOLED`, `NEON_PROJECT_ID`, `PG*`, and `POSTGRES_*` variables. Local `.env*`, `.vercel`, and `.neon` metadata are ignored by Git.

## Deploy and verify

From Ubuntu WSL:

```sh
source /home/barry/.nvm/nvm.sh
nvm use --silent 26
cd /home/barry/src/umami
NODE_OPTIONS=--dns-result-order=ipv4first vercel deploy --prod --yes --archive=tgz --logs --scope caboosesws-projects
```

Verify the stable alias and deployment state:

```sh
curl -I https://umami-beige-eta-20.vercel.app
vercel inspect https://umami-beige-eta-20.vercel.app --scope caboosesws-projects
```

Expected results are HTTP 200 and Vercel status `Ready`. The build checks the Neon connection and applies any pending Prisma migrations. On the initial deployment, all 26 migrations completed successfully.

## Account security

The initial Umami administrator is `admin` with the upstream temporary password `umami`. Change it immediately after the first successful deployment and enable two-factor authentication. Do not record the replacement password in this repository.

## Maintenance

Before taking upstream updates, create a focused branch, fetch `upstream`, review the release and migration notes, merge or rebase the selected upstream revision, and deploy a preview first. Confirm the preview, database migrations, login, and tracking endpoint before promoting it to production.

The WSL checkout is the reusable working environment. Windows Codex remains installed separately; WSL contains native Node/npm, Codex CLI, GitHub CLI, Vercel CLI, Neon CLI, and Cloudflare CLI. Credentials live in each tool's user-scoped credential store, not in this repository.

## Known follow-ups

- Authorize the Vercel GitHub App for `caboosesw/umami`, then connect the repository to enable automatic preview and production deployments. CLI deployments work now.
- Choose the final analytics hostname and add it to the Vercel project, then create the matching Cloudflare DNS record.
- GitHub Issues are currently disabled on the fork, so no deployment-bootstrap issue could be created under the global issue workflow.
