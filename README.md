# Panel Bot 

Discord bots tend to collect a lot of little rules. This project gives them a control panel, so changing a reply, adjusting access, or checking whether a bot is online doesn’t mean digging through command code.

## What it can do

- **Manage bot profiles.** Start and stop bots, check their connection status, and update their Discord presence.
- **Build slash commands.** Create text or embed replies, command options, cooldowns, prerequisites, follow-up messages, and alternative responses.
- **Add interactive replies.** Attach buttons and select menus, with their own replies, access rules, cooldowns, private responses, or DMs.
- **Set up phrase-triggered responses.** Configure a trigger and its replies, including follow-up messages, embeds, buttons, and menus.
- **Send messages from the panel.** Choose a server, channel, or member; save drafts and presets; and review sent-message history.
- **Control access.** Maintain a Discord user whitelist and set the message shown to people who aren’t allowed to use the bot.
- **See the bot’s servers.** Check which guilds the selected bot can access.
- **Run optional Discord API actions from commands.** Actions are limited to supported Discord server and channel routes.

Command changes are saved in the database, but they are not pushed to Discord automatically. Use the registration/sync action when you want Discord to receive the current slash-command definitions.

## How it’s put together

Most of the code is **TypeScript**—JavaScript with types to make the data and API contracts easier to maintain.

- **Dashboard:** React and Vite, with Wouter for page routing and TanStack Query for API data.
- **Bot and API:** Node.js, Express, and discord.js. The Discord bot runs inside the API service.
- **Storage:** PostgreSQL with Drizzle ORM for commands, profiles, presence, message presets, and other settings.
- **Shared API contract:** `lib/api-spec/openapi.yaml` defines the API. Generated React Query hooks and Zod schemas live in the shared `lib/` packages.
- **Workspace:** pnpm keeps the dashboard, API, database code, and shared API packages together.

The dashboard talks to the API, the API reads and writes PostgreSQL, and the bot service uses discord.js to communicate with Discord.

## Project layout

```text
artifacts/
  dashboard/       React dashboard
  api-server/      Express API and Discord bot
lib/
  api-spec/        OpenAPI source contract
  api-client-react/ generated React Query client
  api-zod/         generated Zod schemas and types
  db/              Drizzle database connection and schema
```

## Getting started

You’ll need Node.js 24, pnpm, and a PostgreSQL database.

1. Install dependencies:

   ```sh
   pnpm install
   ```

2. Set these environment variables through your host’s secrets manager:

   - `DATABASE_URL` — PostgreSQL connection string.
   - `SESSION_SECRET` — used to encrypt bot credentials stored in the database. Keep it private and stable; changing it can make previously saved credentials unreadable.
   - `DISCORD_BOT_TOKEN` — optional fallback for the primary bot. You can also enter the token in the dashboard.

3. Create or update the development database schema:

   ```sh
   pnpm --filter @workspace/db run push
   ```

4. Start the API and dashboard as separate services:

   ```sh
   pnpm --filter @workspace/api-server run dev
   pnpm --filter @workspace/dashboard run dev
   ```

This repository is set up around Replit’s path-based routing: the dashboard is served at `/`, and the API at `/api`. Replit’s configured workflows provide the service ports and route requests between them. When running it on a different host, configure an equivalent reverse proxy and set `PORT` for each service.

## Configuration ZIPs

Some packaged copies include `exports/configuracao-painel.json`. The panel does not load that file on its own. If you’re using a ZIP with the optional import helper, follow `README-EXPORTACAO.md`; the importer only runs against an empty database and does not overwrite existing settings.

The saved-configuration export intentionally leaves out bot tokens and webhook URLs. You’ll need to add your own credentials after importing.

## A note about access

There is no sign-in screen or login protection in the dashboard or API. Run it only in a trusted environment or put it behind access controls; anyone who can reach the panel can manage its bots and send messages. Never commit `.env` files, bot tokens, database credentials, or other secrets to GitHub.
