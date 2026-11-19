# How the OS works, and why it is built this way

Read this once before you start. Everything in the kit follows from it.

## The idea in one paragraph

One folder of plain markdown files is your second brain. Every AI you use
(Claude, ChatGPT, Gemini, Claude Code, Codex) reads it before answering about
your work and writes back to it when something happens. On the Mac the AIs read
the files directly. On the web and the phone they reach the same files through
one connector, served by Baalda on your own server. Small jobs keep it clean
by themselves: a daily update, a weekly cleanup, a weekly brief that ends in a
numbered approve list you answer in one line.

## The pieces

| Piece | What it is | Why |
|---|---|---|
| The vault | `~/Vaults/OS`, plain `.md` files in a fixed folder tree | Tool-agnostic. Any AI and any editor can read it, and it survives any tool dying |
| Baalda | A markdown app (Mac + server) that syncs the vault and exposes it as an MCP connector | One sync engine, and a connector so web and phone AIs can read and write with the Mac asleep |
| Your server (optional) | Baalda's server on your own VPS with Coolify, or Baalda's hosted plan | Self-hosted: your notes do not live on someone else's cloud. Hosted: nothing to run. Both give the same connector |
| `AGENTS.md` | The contract every AI reads first, under 200 lines | One rulebook for every AI. `CLAUDE.md` only imports it |
| `MAP.md` | Where everything lives, inside and outside the vault | An AI that has to guess a path will guess wrong |
| `Context/` | 5 to 10 files about you and your businesses, filled by an interview | Ben's tip 1: these files are 80% of the value |
| The instruction boxes | One master text in the vault, pasted into each AI's settings | Every AI behaves the same and knows where the OS is |
| Guarded git backup | Hourly snapshot to a private GitHub repo that refuses a gutted vault | Sync engines lose files. Git is the real rollback |
| Cloud routines | Daily update, weekly cleanup, weekly brief, running on claude.ai | The OS stays current with the laptop closed |
| Skills | Ben's optimizer and operator, plus your own | Method lives in skills; facts live in the OS |

## The rules that matter most

1. **Two kinds of notes.** Current pages (each project's `README.md`,
   `Context/`, decisions, the project register) are always true today and are
   edited in place. Logs (`Daily/`, `log/` folders, `0 Inbox/`) are append-only
   history and get archived. An AI answering "where does X stand?" reads the
   current page, never a pile of logs.
2. **One fact, one home.** A fact is written once, on its current page; every
   other place links to it. Copies drift, and drift makes the vault lie.
3. **Insights, not raw data.** The vault gets decisions, outcomes, corrections
   and things that need you. Commits, test counts and PR rounds stay in GitHub.
   In the original vault, coding agents wrote 657 progress lines into Daily
   notes in one month and buried every decision.
4. **Skills hold method, the OS holds facts.** A copywriting skill keeps its
   frameworks but reads your brand, audience and prices from `Context/`. Change
   the brand once and every skill follows.
5. **Fixed beats perfect.** Lock the folder tree on day one and never rename it.
   Every AI's routing depends on it.
6. **Write it when it happens.** Sessions end without warning. A decision gets
   logged the moment it is made, not at the end of the chat.
7. **Mark what you guessed.** `confirmed`, `MEDIUM`, `LOW`. A vault that guesses
   confidently is worse than one with honest gaps.
8. **One sync engine.** Never put the vault in Google Drive, Dropbox or
   `~/Documents` (iCloud) while Baalda also syncs it. Two engines on one folder
   is how Ben's own vault lost 1,894 notes.
9. **Move notes only inside Baalda.** Baalda tracks each note by an ID. A file
   moved by Finder, a script or a coding agent leaves the original behind and a
   duplicate in the new folder (tested). Creating and editing files is safe.
10. **Act for the business.** Every AI is told to push back when a request
    works against the goals in `Context/`.

## The rhythm once it runs

| When | What runs | What you do |
|---|---|---|
| Any session | The AI reads `AGENTS.md` and the project page, writes outcomes back the same turn | Work normally. Say "log this" when something matters |
| Every hour (Mac) | Guarded git backup | Nothing |
| Every morning (Mac) | Capture of yesterday's coding sessions into `Daily/<date>-capture.md` | Nothing |
| Every morning (cloud) | Daily update: fills gaps in yesterday's Daily note and each project's "Auto (unverified)" block, lists what needs you | Glance at "Needs you" |
| Weekly (cloud) | Cleanup with Ben's optimizer: health score, contradictions, stale pages, up to 10 small fixes | Nothing |
| Weekly (cloud) | Brief: what moved, what stalled, the cleanup's numbered approve list | Answer "approve all" or "all but 3 and 5", about 5 minutes |

## Lessons from the original build (so you skip them)

The first version ran for two months and got messy. The audit found the folder
structure was fine. What broke:

- **Nothing digested.** Agents only appended. One project page reached 709
  lines of dated logs; every project said `status: active`; the register
  disagreed with reality on six projects. Fix: current pages plus the weekly
  cleanup.
- **The machinery reported green while broken.** Only the hourly backup really
  worked. macOS blocked the scheduled scripts from reading the vault, one job
  was never installed, the health check had been stale for weeks. Fix: every job
  writes a heartbeat file, and the health check turns red when a heartbeat is
  older than its schedule.
- **The backup copied a half-loaded vault** (359 files "deleted", restored the
  next morning). Fix: the backup refuses to commit if more than 15 notes vanished
  or any note became empty.
- **Two sync engines** (Google Drive and Baalda) on the same files, and Baalda's
  database (`.context/`) committed to git every hour, 189 MB of dead weight.
  Fix: one engine, and the backup excludes `.context/`.
- **`Context/` stayed draft** with 16 open `(?)` for two months. Fix: the
  interview, one question at a time, by voice.
- **Skills carried their own copies** of brand and audience (one had a 181-line
  copy of the brand). Fix: pointers to `Context/` files.

## Where this comes from

The method is Ben AI's "AI second brain" (two YouTube videos, his PDF guide,
his Notion pages and his three Baalda skills: setup, operator, optimizer). This
kit is that method adapted for a solo operator who already uses several AIs,
with a self-hosted server and the hardening above. Details and links:
`skills.md`.
