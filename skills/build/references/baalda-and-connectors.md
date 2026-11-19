# Baalda on your own server, and every AI connected to it

## What Baalda is

A local-first markdown app (https://baalda.com, source
https://github.com/naveedharri/baalda, Apache-2.0). Notes stay plain `.md`
files on your disk. A small server keeps every device in sync and exposes the
vault as an **MCP connector** at `/api/mcp`, so claude.ai, ChatGPT and Gemini
can read and write your notes from the web or the phone with the Mac asleep.

- Desktop app: Tauri (Mac). Each device keeps a local index (search, backlinks).
- Server: Node on port 3010 (HTTP + WebSocket sync), Better Auth accounts,
  vaults with per-folder permissions.
- Database: Postgres 16. It stores encrypted-looking binary sync updates and
  attachments (25 MB cap per file), not your markdown as text.
- Routes worth knowing: `/api/mcp` (connector, OAuth), `/health`, `/sync`,
  `/p/<token>` (notes you chose to "Publish to the web").
- Inside your vault folder Baalda keeps `.context/` (its local database and
  file registry). **Never edit, move or back it up with another tool.**

## Pick your path

| Path | Effort | When |
|---|---|---|
| **A. Baalda's hosted plan** | 10 minutes | Start here if you have no server. The connector works the same. You can move to your own server later with a fresh import |
| **B. Self-hosted on Coolify** (the original build) | One evening | You already run a VPS with Coolify, or want your notes on your own machine |

No server yet, or a server without Coolify? Do `server-from-zero.md` first: it installs Coolify, because the compose file below relies on Coolify (its network, generated passwords and proxy). Running Baalda without Coolify is possible with upstream's `docs/DEPLOY.md`, but this kit does not cover it.

Whichever you pick, the Mac side and the connectors are identical.

---

## Path B: Coolify, step by step

Ask Claude to do the server steps with you. Claude must confirm each
destructive or public step with you first (DNS record, deploy, opening the app
to the internet). You type every password yourself.

### 1. Your own copy of the code
1. Create a **private** GitHub repo, e.g. `<you>/baalda-os`.
2. Copy upstream at a pinned release tag into it (pick the latest tag on
   https://github.com/naveedharri/baalda/tags):
   ```bash
   T=$(mktemp -d) && git clone -q --depth 1 --branch vX.Y.Z https://github.com/naveedharri/baalda.git "$T/up"
   rsync -a --exclude .git "$T/up/" ./
   ```
3. Add three files of your own: `docker-compose.yaml` (below),
   `patches/invite-only-signup.patch` (below), and `DEPLOY-NOTES.md`: a copy of
   step 10 "Updating Baalda" below, plus this server's specifics (Coolify
   project name, domain, release tag in use). Apply the patch and commit.

Why a private copy and a pinned tag: the server never changes by surprise, and
your patch lives next to the code.

### 2. DNS
An **A record** `os.<yourdomain>` pointing at your server's public IP,
**DNS only (grey cloud)** if you use Cloudflare. Grey cloud lets the server's
own proxy (Traefik) get the Let's Encrypt certificate.

### 3. The compose file (`docker-compose.yaml` at the repo root)

Based on upstream's `deploy/coolify/docker-compose.yml`, with four changes,
each learned the hard way:
1. **No `SERVICE_FQDN_SERVER`** and no domain set in Coolify's UI: Coolify would
   add its own router. Routing is done with the Traefik labels below.
2. **Postgres service renamed** to `baalda-os-postgres`. The server joins
   Coolify's shared `coolify` network, where the name `postgres` resolves to
   *other apps'* databases. With upstream's name the server crash-looped 32
   times on auth errors. This is the biggest trap on a shared Coolify host.
3. A `backups` volume on Postgres for the nightly dump.
4. `SIGNUP_INVITE_ONLY: "1"` (needs the patch below).

```yaml
name: baalda
x-server-image: &server-image
  build:
    context: .
    dockerfile: app/apps/server/Dockerfile
x-server-env: &server-env
  NODE_ENV: production
  BAALDA_DEPLOYMENT: self-hosted
  DATABASE_URL: postgres://baalda:${SERVICE_PASSWORD_64_POSTGRES}@baalda-os-postgres:5432/baalda
  JWT_SECRET: ${SERVICE_REALBASE64_64_JWT}
  BETTER_AUTH_URL: https://os.<yourdomain>
  PORT: "3010"
  SIGNUP_INVITE_ONLY: "1"
  SMTP_URL: ${SMTP_URL:-}
  EMAIL_FROM: ${EMAIL_FROM:-}
services:
  baalda-os-postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: baalda
      POSTGRES_PASSWORD: ${SERVICE_PASSWORD_64_POSTGRES}
      POSTGRES_DB: baalda
    volumes:
      - baalda_pgdata:/var/lib/postgresql/data
      - baalda_backups:/backups
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U baalda -d baalda"]
      interval: 5s
      timeout: 5s
      retries: 20
  migrate:
    <<: *server-image
    command: ["node", "dist/db/migrate.js"]
    environment: *server-env
    depends_on:
      baalda-os-postgres:
        condition: service_healthy
    restart: "no"
  server:
    <<: *server-image
    environment: *server-env
    expose:
      - "3010"
    networks:
      - default
      - coolify
    depends_on:
      migrate:
        condition: service_completed_successfully
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "node", "-e", "fetch('http://127.0.0.1:3010/health').then(r=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))"]
      interval: 15s
      timeout: 5s
      retries: 5
      start_period: 30s
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=coolify"
      - "traefik.http.services.baalda.loadbalancer.server.port=3010"
      - "traefik.http.routers.baalda.rule=Host(`os.<yourdomain>`)"
      - "traefik.http.routers.baalda.entrypoints=https"
      - "traefik.http.routers.baalda.tls=true"
      - "traefik.http.routers.baalda.tls.certresolver=letsencrypt"
      - "traefik.http.routers.baalda.service=baalda"
volumes:
  baalda_pgdata:
  baalda_backups:
networks:
  coolify:
    external: true
```

What the variables are:
- `SERVICE_PASSWORD_64_POSTGRES`, `SERVICE_REALBASE64_64_JWT`: Coolify "magic
  variables". Coolify generates them once and keeps them. The `_64` password
  has no symbols, so it is safe inside the database URL.
- `JWT_SECRET` signs logins and sync tokens. Rotating it logs everyone out.
  Copy it once into your password manager or Keychain so a restore is possible.
- `BETTER_AUTH_URL`: your public address. Used in sign-in and invite links.
- `SMTP_URL` / `EMAIL_FROM`: optional, see step 7. Both empty = email off.
- Coolify gotchas: it treats `${VAR:?text}` as a default value, not an error;
  an unset `${VAR}` arrives as an empty string. Always write `${VAR:-default}`.

### 4. The invite-only patch (`patches/invite-only-signup.patch`)

Upstream lets anyone who finds your server create an account. This patch makes
the server invite-only: the very first user (you, the admin) and anyone with a
pending, unexpired email invitation can sign up; everyone else is refused
before the account exists. It covers email and Google sign-up. Join-code
sign-ups are refused too, so invite people by email.

```diff
diff --git a/app/apps/server/src/auth/auth.ts b/app/apps/server/src/auth/auth.ts
--- a/app/apps/server/src/auth/auth.ts
+++ b/app/apps/server/src/auth/auth.ts
@@ -236,6 +236,31 @@ export const auth = betterAuth({
   },
   databaseHooks: {
     user: {
+      // SIGNUP_INVITE_ONLY=1 makes this server invite-only. A new account
+      // (email or Google) is created only for the very first user (the admin)
+      // or for an address with a pending, unexpired invitation.
+      create: {
+        before: async (user) => {
+          if (process.env.SIGNUP_INVITE_ONLY !== "1") return;
+          const email = String((user as { email?: unknown }).email ?? "").trim().toLowerCase();
+          const { rows } = await authPool.query<{ allowed: boolean }>(
+            `SELECT NOT EXISTS (SELECT 1 FROM "user")
+                 OR EXISTS (SELECT 1 FROM invitation
+                             WHERE lower(email) = $1
+                               AND status = 'pending'
+                               AND "expiresAt" > now()) AS allowed`,
+            [email],
+          );
+          if (!rows[0]?.allowed) {
+            throw new APIError("FORBIDDEN", {
+              message: "This server is invite-only. Ask the owner for an invitation.",
+              error: "signup_invite_only",
+            });
+          }
+        },
+      },
       update: {
         before: async (user) => {
```

Apply with `git apply --3way patches/invite-only-signup.patch`. The line
numbers were written against v0.1.112, where `authPool` and `APIError` are
already imported in `auth.ts`; if you re-create it by hand on a newer release,
check those two imports still exist. If it does not apply to a newer
release, have Claude re-create the same check by hand in
`app/apps/server/src/auth/auth.ts` under `databaseHooks.user.create.before`,
then build and test.

**Rule: never deploy a public server without both the patch and the flag.**
Create your own account first, right after the first deploy, so you are the
first user.

### 5. The Coolify app
- New resource > **Docker Compose**, source = your private GitHub repo (through
  Coolify's GitHub App), branch `main`, base directory `/`, compose file
  `/docker-compose.yaml`.
- Do not set a domain in Coolify. Deploy. `migrate` runs first, then `server`.
- Check: `curl https://os.<yourdomain>/health` returns JSON.

### 6. Prove it is locked
From outside (your phone on mobile data, or ask Claude to run it from a cloud
session): try to sign up with a made-up address. It must be refused with
`signup_invite_only`. If it succeeds, take the server offline and fix the patch
before anything else. Delete any test account that slipped through.

### 7. Email (optional, recommended)
Without email, invites are shared as links and "Forgot password" does not work.
Any transactional SMTP provider works (the original build used emailit with a
"sending only" key scoped to the domain).
- `SMTP_URL` = `smtp://user:pass@host:587` or `smtps://user:pass@host:465`
  (you paste it into Coolify yourself; never into a chat)
- `EMAIL_FROM` = `os@<yourdomain>` (add the provider's SPF/DKIM records first)
- **Trap:** the server refuses to start if `EMAIL_FROM` is set but `SMTP_URL`
  does not reach the container. In Coolify, make sure the variable is set for
  **Production**, not preview only. (This caused 11 minutes of downtime.)

### 8. Nightly database dump
Coolify > the app > Scheduled tasks > new task, container `baalda-os-postgres`,
cron `0 1 * * *` (UTC), timeout 600:
```bash
pg_dump -U baalda -d baalda -Fc -f /backups/baalda-$(date +%F).dump && find /backups -name 'baalda-*.dump' -mtime +14 -delete
```
Then copy the dumps **off the server** (a weekly pull to the Mac, or the
server's own backup to S3). The original build left them on the server, which
is a gap: if the server dies, the dumps die with it. Your markdown is still safe
in the git backup, but sharing settings and history are not.

### 9. Public or private?
The first version allowed the full app only from Tailscale addresses and opened
just the connector and sign-in paths to the internet. It was later opened fully
so the owner could invite people as viewers or editors. What keeps it safe:
invite-only sign-up plus Baalda's own login and per-vault access on every note.
Public with the patch is the simpler setup; Tailscale-only is stricter if you
never invite anyone. If you go Tailscale-only, your Mac must reach the server
by its Tailscale address (an `/etc/hosts` line mapping `os.<yourdomain>` to the
server's Tailscale IP), and remember that line exists if you later go public.

### 10. Updating Baalda (server first, always)
1. Run the nightly dump task now and confirm the file exists.
2. Copy the new upstream tag over your repo, keeping your own files:
   ```bash
   T=$(mktemp -d) && git clone -q --depth 1 --branch vX.Y.Z https://github.com/naveedharri/baalda.git "$T/up"
   rsync -a --delete --exclude .git --exclude docker-compose.yaml --exclude DEPLOY-NOTES.md --exclude patches "$T/up/" ./
   git apply --3way patches/invite-only-signup.patch
   ```
3. Read upstream's CHANGELOG for breaking changes. Commit "Baalda vX.Y.Z", push.
4. Redeploy in Coolify. Check `/health` and that a made-up sign-up still answers
   `signup_invite_only`.
5. **Only then** update the Mac app. A newer app on an older server switches
   features off.

---

## The Mac side (all paths)

1. **Make a backup first.** If you already have notes, archive the folder
   somewhere that is not synced (`tar -czf ~/os-backup-$(date +%F).tgz <folder>`).
2. **Pick the folder:** `~/Vaults/OS`. Not `~/Documents` (often synced to
   iCloud), not Google Drive or Dropbox. One sync engine only.
3. Install the Baalda Mac app from https://baalda.com. Account menu > Server
   settings > `https://os.<yourdomain>` > Save (skip for the hosted plan).
4. Create your account (your email, your password, your hands). As the first
   user you are the admin.
5. **Rehearse on a throwaway vault first** (10 minutes, worth it): create a
   vault in a new folder `~/Vaults/rehearsal`, link it to the server, then from Finder or Claude Code (a) edit a
   note, (b) rename a note in the same folder, (c) move a note to another
   folder. What the original build found: edits and same-folder renames sync
   fine and keep the note's identity; **a move to another folder done outside
   the app leaves a duplicate**. That is why AGENTS.md says moves happen only in
   Baalda or through the connector's `move_note` / `move_folder`. Delete the
   rehearsal vault afterwards.
6. Open `~/Vaults/OS` in Baalda and turn on sync. Check: every top-level folder
   shows in Baalda's sidebar, and three notes you pick at random open with
   their content. (The exact count is checked in Phase 4 through the connector:
   `list_notes` must return as many notes as `find ~/Vaults/OS -name '*.md' -not -path '*/.context/*' | wc -l`.)
7. Settings > General > **Freeze vault root**: nothing new can be created at the
   top level, by anyone, until you lift it. This enforces "never add a
   top-level folder" on every AI.
8. Claude Code and Codex on the Mac edit the files directly; Baalda's watcher
   syncs them. They need no connector. Make the vault a standing folder:
   - Claude Code, `~/.claude/settings.json` (inside `permissions`, not top level):
     ```json
     { "permissions": { "additionalDirectories": ["/Users/<you>/Vaults/OS"] } }
     ```
   - Codex, `~/.codex/config.toml`:
     ```toml
     [projects."/Users/<you>/Vaults/OS"]
     trust_level = "trusted"
     ```
     (check current Codex docs; key names change between versions)

## Connect every AI

Use the same connector name, **OS**, everywhere, so "use the OS connector"
means the same thing in every AI.

**claude.ai** (also covers the Claude phone app and cloud routines)
Settings > Connectors > Add custom connector. Name `OS`, URL
`https://os.<yourdomain>/api/mcp`, leave advanced OAuth settings empty. A Baalda
page opens: sign in, choose vault **OS**, Allow access.

**ChatGPT**
Plugins (or Apps / Connectors, depending on your plan) > Add > Add custom MCP
server. Name `OS`, Connection = Server URL `https://os.<yourdomain>/api/mcp`,
Authentication **OAuth** (leave advanced settings alone). Tick "I understand"
> Create. On the Baalda page choose vault OS > Allow.

**Gemini**
Customize > Connectors > + Custom: URL `https://os.<yourdomain>/api/mcp`, sign
in, choose vault OS, Allow. Gemini names it **"Context"** (Baalda's internal
name): rename it to **OS** on the "Save your custom app" screen.

**Claude Code / Codex as connector (optional)**
Not needed on the Mac (they read files). For a cloud or remote coding agent,
Baalda can issue a bearer token in Vault settings > MCP. The person creates it
and types this command themselves in Terminal, replacing `<token>` (Claude
never sees the token):
`claude mcp add --transport http os https://os.<yourdomain>/api/mcp --header "Authorization: Bearer <token>"`.
Keep the token out of any repo.

**The test, in each AI, in a new chat:**
"Use the OS connector: read AGENTS.md and tell me its first heading."
Then the real one, from the phone with the Mac lid closed: "Use the OS
connector: add a line to today's Daily note: phone test passed." Open the Mac
later and check the line arrived.

## Gotchas collected

- The connector answers `405` to a plain GET on `/api/mcp`; that is normal (it
  expects POST).
- `/health` may show an internal version (`0.1.0`), not the release tag. Check
  the release some other way when updating.
- The Mac app must be at least the server's `minDesktopVersion` (in `/health`).
- If the old vault used Baalda's hosted plan, use "Make local only" in the app
  before moving to your own server; that deletes the hosted copy and keeps your
  files.
- Never commit `.context/` to the git backup (one build committed 189 MB of it).
