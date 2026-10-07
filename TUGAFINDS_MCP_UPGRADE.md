# TugaFinds MCP upgrade

The local `codex/enable-mcp` branch is based on the official Umami `v3.4.0` release, commit `ec0ff50388c264ed8ce46f00967e92f7e71476ae`.

Production currently uses the fork's `master`, commit `c0ea3aefbee7a3429ee2f824b06dc4a9dbe0b7e1`, version 3.1.0.

## Release target

- Vercel project: `umami`, `prj_zNqbt5FGkvAXX7GTaTbTenyNYqvn`.
- Public URL: https://umami-five-sand.vercel.app/
- TugaFinds website: `f871d6f7-da3a-468b-b07f-85a9fe13d276`.
- Required Umami setting: `MCP_ENABLED=1`.
- MCP endpoint after release: https://umami-five-sand.vercel.app/mcp
- Credentials: separate `umami_` API keys for Codex and Grok Bot, created after the database upgrade.

## Database migration impact

The upgrade adds migrations 20 through 26. Migration 20 renames the website recorder setting. Migration 23 removes older duplicate rows for each session/property pair before adding a unique index. Migration 24 normalizes usernames. Migration 26 creates the API-key table required by remote MCP authentication.

The build automatically runs these migrations unless `SKIP_DB_CHECK=1` skips the whole database check, or `SKIP_DB_MIGRATION=1` skips only migrations. These flags are for preview preparation; the live upgraded app needs the new schema.

Production and preview have a DATABASE_URL configured. Before an upgrade preview, use an isolated database or set a branch-scoped preview-only `SKIP_DB_CHECK=1`. Before the production upgrade, preserve a restorable database backup and approve the migration that removes duplicate property rows. Do not publish an upgrade preview that can migrate production as a side effect.

Rolling back Vercel's application deployment does not reverse database migrations. Restore the database backup as part of a full rollback if needed.

## Validation after release

1. Sign in and verify the existing TugaFinds website and historical self-hosted traffic.
2. Confirm `/script.js` and `/api/heartbeat` return 200.
3. Confirm unauthenticated `/mcp` requests are rejected.
4. Create a key for each client in Settings > API keys, then connect both clients.
5. Call `list_websites` and query TugaFinds visitor/pageview statistics.
6. Confirm a pageview and listing click from the deployed TugaFinds tracker appear in Umami.

See https://docs.umami.is/docs/mcp for the official MCP setup and supported read-only tools.
