# Ben's method, his skills, and your own skills

## The public sources (free)

| What | Link |
|---|---|
| Video 1: "Build a Better Claude Second Brain than 99%" | https://www.youtube.com/watch?v=Puzgmzv7t3Q |
| Video 2: "18 Tips to Set Up Your Ultimate AI Second Brain" | https://www.youtube.com/watch?v=pXHpUyC54uQ |
| Guide page (18 tips, PDF guide, setup skill) | https://benai.notion.site/ai-second-brain-guide |
| Baalda page (the Baalda versions of setup, optimizer, operator, and the five model rules) | https://benai.notion.site/Fable-5-1-Second-Brain-3c81124570fe802f9e09dfac729373c9 |
| Baalda | https://baalda.com |

Watch video 2 first (the 18 tips), then video 1 (Baalda and the optimizer).
Download the skills from the Baalda Notion page, not from anyone's copy: they
are Ben's work and he updates them.

## The 18 tips, one line each, and where the kit covers them

| # | Tip | In this kit |
|---|---|---|
| 1 | Brain dump: a voice interview makes 5-10 context files | `interview.md` |
| 2 | Keep it local: plain `.md` in one folder | `~/Vaults/OS` |
| 3 | Make it visual: Obsidian or Baalda as the viewer | Baalda |
| 4 | `AGENTS.md` is a map under 200 lines; not two root rule files | `CLAUDE.md` only imports `AGENTS.md` |
| 5 | One second brain (split only personal vs business if you must) | One vault |
| 6 | Lock the folder structure: fixed beats perfect | Rule in `AGENTS.md`, plus Baalda's "Freeze vault root" |
| 7 | An index file in every folder | `README.md` in every folder with more than 10 notes |
| 8 | Save insights, not raw data | Code proof stays in GitHub; Daily lines only for decisions, shipped, corrections, needs-you |
| 9 | Daily update | Routine 1 in `automation.md` |
| 10 | Weekly clean-up with a health score | Routine 2, Ben's optimizer |
| 11 | Dashboard | Optional, later: only once the pages it reads are true |
| 12 | Skills read from the OS | Below |
| 13 | Runs with the laptop closed | Cloud routines plus the connector |
| 14-17 | Team sync, staged rollout, permissions, team dashboards | Later. Baalda has per-folder access; invite people when you are ready |
| 18 | Business-first rule | One line in `AGENTS.md` and in every instruction box |

## The five model rules (from video 1)
1. Search the vault before answering about a name you do not know confidently,
   using the name exactly as written.
2. Edit surgically: change one line, do not rewrite the note.
3. Quote properly: one worked example in `AGENTS.md` beats a rule.
4. Read in batches: list what you need, then fetch everything independent at once.
5. Medium effort for everyday notes work; high for audits and plans.
All five are already in the `AGENTS.md` template. The Notion page has
Anthropic's exact wording if you want it.

## Ben's three skills, and how to use them safely

Install: download each zip, then either unzip into `~/.claude/skills/<name>/`
(Claude Code) or upload under claude.ai > Settings > Capabilities > Skills.

| Skill | What it does | How to use it here |
|---|---|---|
| **setup** (`/second-brain-setup`) | Builds a vault: folders, per-folder index files, the interview, Context files | **With this kit you do not need it:** `interview.md` covers the brain dump and the kit builds the folders. If you use it anyway, never let it build folders inside a vault that already exists. **Warning:** it triggers on everyday words like "set up" and "initialize", and if a root `CLAUDE.md` exists it offers a full reset that deletes everything. Once your vault exists, disable or remove this skill |
| **optimizer** | Ten checks (instruction-file quality, wiki structure, compression, context rot, memory hygiene, skill structure, general hygiene, reflection, architecture, rules written for an older model) and a health score out of 100 | First run in **"save everything to plan"** mode: it changes nothing and becomes your audit. Then weekly as Routine 2, with house rules: at most 10 mechanical fixes alone, everything else on a numbered approve list, its HTML report kept out of the vault, never a new top-level folder. Take a backup before it applies anything |
| **operator** (daily updater) | Asks what it cannot infer, then renders a ~400-line daily-update prompt and schedules it | Run it once to generate a draft, then strip three things that clash with this kit: the task list it keeps inside the vault (tasks live in your task tool), the coloured signature it stamps on every file, and the folder it creates for itself. Or skip it and use the Daily Update prompt in `automation.md`, which is already adapted |

Ben's older bundle also had Obsidian-era extras (a Relay MCP server on
Railway, an Obsidian dashboard, a team-permissions plugin). Baalda's own
connector and permissions replace all three; you do not need them.

## Skills read from the OS (tip 12)

**Skills hold method, the OS holds facts.** A copywriting skill keeps its
frameworks, platform specs and checklists. It does not keep your brand,
audience, offer, prices, voice, people or clients: it reads them from
`Context/` every time. Change the brand once, and every skill follows.

For every skill you own (in `~/.claude/skills/`, in claude.ai, in Codex):
1. Split its reference files in two: craft stays, business facts leave.
2. Before deleting a fact copy, merge anything it knows that `Context/` does
   not into the right Context file (one skill in the original build carried a
   181-line copy of the brand, some of it contradicting `brand.md`).
3. Replace the copy with a short pointer, for example:
   ```
   Before you start, read from the user's OS (the "OS" connector, or ~/Vaults/OS on the Mac):
   Context/brand.md, Context/icp.md, and the project README the task is about.
   If you cannot reach the OS, say so and stop. Never fall back to what you remember.
   ```
4. Edit each skill once at its source, then re-upload or sync it.
5. The weekly cleanup checks every skill for business facts that crept back.

**Test:** change one line in `Context/brand.md`, run the skill in a fresh
session, and the output reflects the change.

## Two small skills worth writing yourself (ask Claude to make them)
- **new-project**: runs the new-project checklist in
  `3 Resources/playbooks/new-project.md` (register check, folder, templates,
  register row, link block, Daily line, verify).
- **wrap-up**: at the end of a working session, writes what changed into the
  project README, decisions and the Daily note, moves task cards to Done, and
  says what is still open.
Both should point at the vault files for their rules rather than copy them.
