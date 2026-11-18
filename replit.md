# Discord Bot Dashboard

A web dashboard for controlling a discord.js bot (start/stop, view status/metrics/guilds) and managing its slash commands (create/edit/delete, text or embed replies, cooldowns) without touching code.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server + bot (port 8080, path `/api`)
- `pnpm --filter @workspace/dashboard run dev` — run the web dashboard
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `DISCORD_BOT_TOKEN` — bot token

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5 + discord.js (bot runs in-process inside the API server)
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)
- Frontend: React + Vite, wouter routing, shadcn/ui, TanStack Query

## Where things live

- Bot lifecycle (start/stop/login) — `artifacts/api-server/src/bot/client.ts`
- Slash command DB repo (CRUD, cooldown tracking) — `artifacts/api-server/src/bot/commandsRepo.ts`
- Dynamic command registration (builds Discord command JSON from DB) — `artifacts/api-server/src/bot/commands.ts`
- Bot routes (status/start/stop/commands CRUD/guilds) — `artifacts/api-server/src/routes/bot.ts`
- DB schema: `bot_commands` (slash commands, text/embed reply config, cooldownSeconds) and `command_cooldowns` (per-user/per-command last-used timestamps) — `lib/db/src/schema/bot-commands.ts`, `command-cooldowns.ts`
- API contract — `lib/api-spec/openapi.yaml`
- Command management UI (create/edit/delete, embed builder with image/thumbnail/color, cooldown picker) — `artifacts/dashboard/src/pages/Commands.tsx`

## Architecture decisions

- Slash commands are **DB-backed, not hardcoded** — admins create/edit commands from the dashboard (name, description, reply type text/embed, embed title/description/color/image/thumbnail, cooldown) instead of editing code. Editing a command in the DB does not push it to Discord automatically; user must click "Sincronizar" (calls `POST /bot/commands/register`) to PUT the full command set to Discord's global command endpoint.
- Per-user cooldowns persist in `command_cooldowns` (not in-memory) so long cooldowns (e.g. 1 year) survive server restarts.
- Bot commands are registered globally (`ApplicationIntegrationType.GuildInstall` + `UserInstall`, all `InteractionContextType`s) so the bot works in guilds, DMs, and as a user-installed app.

## User preferences

- User communicates in Brazilian Portuguese, informal tone. Match that register in responses and in bot-facing strings (log messages, UI copy).

## Gotchas

- Discord enforces a **global IP-level rate limit** independent of per-route limits. Rapid start/stop/register cycles during testing can get the whole sandbox IP blocked (`429` with `retry-after` on `https://discord.com/api/v10/gateway`) for 20+ minutes — `client.login()` just hangs/times out during the block. Check `curl -i https://discord.com/api/v10/gateway` first before assuming a code bug when login hangs.
- Discord desktop/mobile clients cache slash commands locally; after registering/removing global commands, a full client restart may be needed before the user sees the change — this is not a backend bug.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
