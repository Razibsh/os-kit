# Getting a server, from zero

Read this when the person has no server, or a server without Coolify. Skip it
if they already run Coolify.

## First, the honest choice

| Option | Cost | Effort | Pick it when |
|---|---|---|---|
| **Baalda's hosted plan** | Free tier | 10 minutes | They just want it working, or have never used a terminal. Notes sit on Baalda's cloud. They can move to their own server later with a fresh import |
| **Own server with Coolify** | A small VPS, roughly the price of a coffee a month, plus a domain | One evening | They want their notes on a machine they control, or will run other apps too |

Explain both in plain words and let them choose. Do not push the self-hosted
path on someone who would be better served by the hosted plan.

## Option A: Baalda's hosted plan

1. They download the Baalda Mac app from https://baalda.com and create an
   account (their email, their password).
2. Skip the server steps in `baalda-and-connectors.md`. Everything from "The
   Mac side" onward is the same.
3. The connector address is the one Baalda shows in the vault's settings
   (Vault settings > MCP). Use it wherever the kit says
   `https://os.<yourdomain>/api/mcp`.

## Option B: a server with Coolify

### 1. Rent a VPS (they do this, Claude explains)
- Any provider works: Hetzner, DigitalOcean, Vultr, Linode, and so on.
  Coolify's docs ask for **at least 2 CPU cores, 2 GB RAM, 10 GB free disk**,
  amd64 or arm64. Baalda is light; 2 cores and 4 GB is comfortable.
- Operating system: **Ubuntu 24.04 LTS** (or the newest Ubuntu LTS offered).
- They create the account and pay. Claude never handles payment details.
- **SSH key, not a password.** Before creating the server, Claude makes a key
  on the Mac and shows the person the *public* half to paste into the
  provider's "SSH keys" field:
  ```bash
  ssh-keygen -t ed25519 -f ~/.ssh/os-server -C "os-server"
  cat ~/.ssh/os-server.pub
  ```
  The private file `~/.ssh/os-server` never leaves the Mac.
- When the server exists, the person tells Claude its public IP. Claude adds an
  alias so later steps are simple:
  ```
  # ~/.ssh/config
  Host os-server
    HostName <server-ip>
    User root
    IdentityFile ~/.ssh/os-server
  ```
  Test: `ssh os-server 'uname -a'`. Claude confirms the host key fingerprint
  with the one the provider shows, the first time.

### 2. Firewall
In the provider's firewall (or `ufw` on the server), allow only:
`22` (SSH), `80` and `443` (web), and for the first setup only `8000`, `6001`,
`6002` (Coolify's dashboard before it has a domain). Close `8000`, `6001`,
`6002` once the dashboard works through its own domain.

### 3. Install Coolify (one command)
With the person's yes, Claude runs it over SSH:
```bash
ssh os-server 'curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash'
```
Check Coolify's current docs first (https://coolify.io/docs) in case the
command changed. The installer prints the next steps.

**Right after it finishes**, the person opens `http://<server-ip>:8000` and
creates the admin account straight away (their email, their password). Until
that account exists, anyone who finds the address could register first.
(Coolify also supports pre-creating the admin through environment variables
during install. If the person prefers that, they type the command with their
own password themselves; Claude never types it.)

### 4. A domain
- They need a domain they control (any registrar; Cloudflare Registrar and
  Namecheap are common). Claude explains how to add DNS records at their
  registrar.
- Two A records pointing at the server's IP, DNS only (no proxy, "grey cloud"
  on Cloudflare):
  - `coolify.<domain>` for the dashboard
  - `os.<domain>` for Baalda
- In Coolify: Settings > set the instance domain to `https://coolify.<domain>`.
  Coolify gets the HTTPS certificate itself. Then close ports 8000, 6001, 6002.

### 5. Optional: Tailscale
Tailscale is a free private network between your own devices. With it on the
Mac and the server, SSH can be closed to the internet entirely and opened only
on the private network. Recommended for anyone who will keep the server for
years; skip on day one if it feels like too much. Docs: https://tailscale.com/kb.
If they add it, update `~/.ssh/config` to the server's Tailscale address and
then close port 22 publicly.

### 6. Keep it healthy
- Turn on the provider's automatic backups or snapshots if offered (small cost).
- Coolify updates itself from its dashboard; update Coolify before updating
  Baalda.
- Baalda's own update procedure is in `baalda-and-connectors.md`.

Then continue with "Path B: Coolify, step by step" in `baalda-and-connectors.md`.

## Not on a Mac?

| Part | Windows / Linux |
|---|---|
| Vault, interview, rules, templates | Works the same (`C:\Users\<you>\Vaults\OS` or `~/Vaults/OS`) |
| Baalda app | Check https://baalda.com for the current desktop platforms. If there is no app for their system, they can still use the vault with any editor plus the connector, as long as one device runs the Baalda app |
| Server, Coolify, connectors | Same |
| Mac jobs (backup, capture, health) | The scripts are macOS (launchd). On Linux, Claude can port them to `systemd` timers or cron. On Windows, to Task Scheduler with PowerShell, or run them inside WSL. Say so up front and offer to port them |
| Cloud routines | Same |
