# Vault templates

Every file the vault starts with. Claude writes these in Phase 1 of
`SKILL.md`, filling `<placeholders>` from the
interview. Never leave a placeholder in a saved file: an empty section says
"EMPTY ON PURPOSE" and what would fill it.

## The folder tree

```
~/Vaults/OS/
  AGENTS.md            the operating contract, read by every AI, under 200 lines
  CLAUDE.md            one line: @AGENTS.md (so Claude Code loads the same rules)
  MAP.md               where everything lives, inside and outside the vault
  START-HERE.md        the human front door, short on purpose
  0 Inbox/             anything unsorted; the weekly cleanup empties it
  1 Projects/          work with an end date
    _REGISTER.md       every project ever started, and what happened to it
    <slug>/README.md   the current page (true today)
    <slug>/decisions.md
    <slug>/open-questions.md
    <slug>/log/        dated notes: history
  2 Areas/             ongoing, no end date (one folder per business)
    personal/          always created: the routines write os-health.md here and reports in personal/log/
  3 Resources/         rules, prompts, playbooks, reference
    playbooks/
  4 Archive/<year>/    finished or superseded; never read unless asked
  5 Evidence/          optional: one file per finished thing worth showing a client
  Context/             who you are and how the businesses work (the AI briefing pack)
  Daily/               YYYY-MM-DD.md, one line per event
  Notes/
    decisions/         decisions that span projects
    ideas/
    books/
```

Rule: this shape is fixed once created. Never rename a numbered folder and never
add a top-level folder. Every AI's routing depends on it staying put ("fixed
beats perfect").

---

## AGENTS.md

```markdown
# Operating contract - <Your name> OS

<Your name>'s second brain, shared by every AI they use: Claude, ChatGPT,
Codex and Gemini follow the same rules. Human lost? Read `START-HERE.md`. Where
things live: `MAP.md`. Detail lives in `3 Resources/`. This file stays under 200
lines because every AI reads all of it at the start of every session.

## Read first
1. `MAP.md` to find anything. Never guess a location.
2. The current page for the subject: the project or area `README.md`, then
   `Context/` for who <name> is and how the businesses work. Open `log/` folders
   and `Daily/` only when you need history.
3. Before answering about a project, client, tool or name you do not know
   confidently, search the vault first, using the name exactly as <name> wrote
   it. Partial familiarity is what makes an outdated answer sound right.
4. Need several files? List them first, then read every one that does not
   depend on another in one batch. It is faster and the answer is the same.

## Two kinds of notes
- **Current pages:** each project and area `README.md`, `Context/`,
  `Notes/decisions/`, `1 Projects/_REGISTER.md`. They say what is true today.
  Edit them in place and never leave an old and a new version standing.
  `reviewed:` is set only by <name> or by a change they approved, so it keeps
  meaning "a human checked this".
- **Logs:** `Daily/`, the `log/` subfolder of a project or area, `0 Inbox/`.
  Append only. Old logs move to `4 Archive/<year>/` (Daily after 60 days).

**One fact, one home.** Write a fact on its current page and link to it from
everywhere else. Copies drift apart, and drift is what makes a vault
contradict itself.

## Where to write
| What happened | Where it goes |
|---|---|
| A decision or a reversal | The project's `decisions.md` (or `Notes/decisions/` if it spans projects), plus one Daily line |
| Something shipped, changed or corrected | Edit the current page in place, plus one Daily line ("Corrected <date>: was X, now Y") |
| A handoff, run report, verification, research | A dated note in that folder's `log/`. The README points at `log/` once instead of listing each note |
| Not sure where | `0 Inbox/YYYY-MM-DD-short-slug.md` |
| Your tool can create files but not edit them | A new `0 Inbox/` note with its Daily line inside |
| Ideas, book notes | `Notes/ideas/`, `Notes/books/` |
| A new place things live, or one that moved or closed | Add, fix or remove its row in `MAP.md` in the same turn |

Never add a top-level folder, rename a numbered folder, or run a setup, reset
or reorganise routine on this vault unless <name> asks. Audit skills may run in
report-only mode; applying their changes needs <name>'s numbered approval.
**Exception:** the weekly cleanup routine may apply up to 10 mechanical fixes
per run on its own (a duplicate H1, an obvious missing frontmatter field, a wiki
link whose renamed target exists exactly once, and any punctuation rule this
file sets), and lists them in its report. Everything else waits for approval.

**Moving notes between folders:** only inside the Baalda app or with the
connector's move tool (`move_note` / `move_folder`), never with a file move,
script or shell command. Baalda tracks each note by an ID; a move made outside
it leaves the original in place and a duplicate in the new folder. Creating,
editing and renaming within a folder are safe. Never touch `.context/`
(Baalda's database).

## Daily notes
One line per event, starting with your tag and the time:
`- (claude) 14:05 | ...`. Write a line for a decision, something shipped,
a correction, or something that needs <name>. Code progress (commits, test
counts, PR rounds) stays in the pull request: otherwise hundreds of those lines
bury the decisions.

## Write it when it happens
Sessions end without warning, so log the moment something lands, not at the
end. "Log this" means do it now. Tool memory (Claude, ChatGPT, Codex, Gemini)
is a cache; this vault is the record. Repeated procedures become playbooks in
`3 Resources/playbooks/`.

## What never goes in the vault
- **Binaries or anything over 1 MB.** They go to <cloud drive>/Assets; the vault
  keeps a note with the link. The vault is backed up to git every hour, and a
  binary stays in that history for good.
- **Tasks.** They live in <task tool> (`3 Resources/tasks.md`). Link card and
  note both ways.
- **Code and its proof.** GitHub. The vault keeps the reasoning and the outcome.
- **Copies of business facts inside skills.** Skills hold method. A skill that
  needs brand, prices, audience or voice reads the right `Context/` file, so one
  edit updates every skill.

## How to write
- Frontmatter on every note: `type`, `date`, `status` (one of the eight in
  `3 Resources/status-vocabulary.md`), `tags`, `author`, `why`. Daily notes need
  only `type` and `date`. Editing someone else's note: add `updated` and
  `updated_by`.
- Edit surgically: change only the lines that need it. Rewriting a whole note
  to change one part is slower and is how content gets lost.
- `[[wiki links]]` with the full vault path. Anything outside the vault gets a
  clickable URL.
- Mark claims that are not first-hand from <name> with one word, `confirmed`,
  `MEDIUM` or `LOW` (`3 Resources/confidence-and-claims.md`), only where it
  adds signal.
- When you reuse a note's own words, quote and link them; never present them as
  your summary.
- <Language rule, e.g. "English by default; Hebrew content is native Hebrew, never translated".>

## Starting anything new
1. Read `1 Projects/_REGISTER.md` and search the vault for the name. Already
   there? Say so and update that row and folder; never a second one.
2. Search <task tool> for its open cards and move the active one to Doing.
3. Genuinely new: follow `3 Resources/playbooks/new-project.md` step by step.

## Act for the business
Always act in the best interest of the business, based on the goals and
strategy in `Context/`. If a request or an output conflicts with that, say so
clearly instead of just agreeing.

## Effort
Everyday notes work (reading, logging, small edits) runs well on medium effort.
Keep high effort for audits, plans and hard decisions.

## Who <name> is
<Two or three lines. Everything else: `Context/`.>
```

## CLAUDE.md

```markdown
# <Your name> OS

All rules for this vault live in one file, `AGENTS.md`, so Claude, ChatGPT,
Codex and Gemini follow the same text. Claude Code loads it from here:

@AGENTS.md
```

## MAP.md

```markdown
---
type: map
date: YYYY-MM-DD
status: active
tags: [os]
---
# MAP - where everything lives

The OS itself: `~/Vaults/OS` on the Mac, synced by Baalda (<your server os.yourdomain.com, or Baalda's hosted plan>).
Cloud AIs reach it through the "OS" connector.
If you are an AI reading this: never guess a location. Route from here.

## Inside this vault (markdown only)
| Folder | Holds |
|---|---|
| `0 Inbox/` | Anything unsorted. The weekly cleanup empties it. |
| `1 Projects/` | Active work with an end date. `_REGISTER.md` lists every project ever started. Each project: `README.md` (current page), `decisions.md`, `open-questions.md`, `log/` (history). |
| `2 Areas/` | Ongoing, no end date: <list your areas>. |
| `3 Resources/` | Rules, prompts, playbooks, reference. |
| `4 Archive/` | Finished or superseded, by year. Never read unless asked. |
| `Context/` | Who <name> is, brand, customers, strategy, stack. Treat any `(?)` line as unconfirmed. |
| `Daily/` | `YYYY-MM-DD.md` one per working day; `YYYY-MM-DD-capture.md` written each morning for the day before. |
| `Notes/` | `ideas/`, `books/`, `decisions/`. |

## Outside this vault
| What | Where | Notes |
|---|---|---|
| Images, video, PDFs, exports | <Drive>/Assets/<brand>/<kind>/ | Never in the vault. The vault keeps a note with the link. |
| Code | <~/Work/...> and GitHub | Each repo keeps its own STATUS / DECISIONS |
| Tasks | <Trello board / Todoist / Linear> | Single source of truth for tasks |
| Vault backup | <private GitHub repo> | Hourly guarded snapshot. Never opened by hand. |
| Baalda server | <Coolify project name, or "Baalda hosted plan"> | Self-hosted: update procedure in the server repo's `DEPLOY-NOTES.md` (<repo URL>) |

## Cadence
Mac (needs the Mac awake): hourly guarded backup; daily 06:15 capture of yesterday;
weekly health check.
Cloud routines (run with the Mac off, through the OS connector): daily update;
weekly cleanup; weekly brief. Cron times in routines are UTC: re-check them at
every daylight-saving change.

## The four root files
- `CLAUDE.md` - loads `AGENTS.md` for Claude Code.
- `AGENTS.md` - the operating contract. Binding on every AI.
- `MAP.md` - this file.
- `START-HERE.md` - the human front door.

Hidden tool folders (`.claude/`, Baalda's `.context/`) are allowed and are not notes. Never edit `.context/`.
```

## START-HERE.md

```markdown
---
type: system
date: YYYY-MM-DD
status: active
tags: [os]
---
# Start here

If you are lost, read this file. It is short on purpose.

## The one-sentence version
**Everything you think goes in this vault. Everything you must do goes in
<task tool>. Everything heavy goes in <Drive>/Assets. Code goes in a repo.
Nothing goes anywhere else.**

## Where do I look for something?
| I want to find... | Go to |
|---|---|
| A decision I made, and why | **Ask Claude.** "What did we decide about X?" It reads the vault. |
| What happened on a day | `Daily/YYYY-MM-DD.md` |
| What a project is and where it lives | `1 Projects/_REGISTER.md` |
| A fact about the business, brand, customers | `Context/` |
| How something works, a rule | `3 Resources/` |
| What I have to do | <task tool> |

**Ask Claude first, Baalda second.** Baalda's search matches words; Claude
reads meaning.

## The daily rhythm
- Finished something worth remembering? Tell any AI. It writes the Daily line
  and, for a decision, the project page. You never create these files by hand.
- Have a task? Tell any AI, or add it to the task inbox yourself.
- Want to know where something stands? Ask, or open the project's `README.md`.
- Once a week the brief arrives with a numbered approve list. Answer
  "approve all" or "all but 3 and 5" in any Claude.

## The three rules that keep it from rotting
1. Never invent a new top-level folder. If nothing fits, `0 Inbox/`.
2. Never keep a heavy file in the vault. Drive, with a pointer note.
3. Name new files `YYYY-MM-DD_brand_what-it-is_version.ext`.
```

## 1 Projects/_REGISTER.md

```markdown
---
type: register
date: YYYY-MM-DD
status: active
tags: [projects]
---
# Project register

**Every project ever started, and what happened to it.**

Rule (also in `AGENTS.md`): before helping start anything new, read this file
first. If it is already here, say so and point at the existing work.

| Project | One line | Status | Last seen | Where |
|---|---|---|---|---|
| <name> | <what, for whom> | active | YYYY-MM-DD | [[1 Projects/<slug>/README]] |
```

## Project README.md (the current page, about 60 lines max)

```markdown
---
type: project
date: YYYY-MM-DD
status: active
tags: [projects, <brand>]
reviewed: YYYY-MM-DD
---
# <Project name>

<One line: what it is and for whom.>

## Now
- <What is true today, 3 to 6 bullets, each with a date.>

## Decisions in force
[[1 Projects/<slug>/decisions|decisions]]

## Open questions
[[1 Projects/<slug>/open-questions|open-questions]]

## Where things live
| What | Where |
|---|---|
| Code | <GitHub URL> |
| Files | <Drive folder URL> |
| Live | <https URL> |
| Tasks | <card URL> |

## Next milestone
<One line with a date.>

## Auto (unverified)
_Reserved for the daily update routine. It may only write here, always with a source link._

History: older dated notes are in this folder's `log/` subfolder.
```

## decisions.md and open-questions.md

```markdown
# Decisions - <Project name>

- YYYY-MM-DD: <what was decided>. Why: <reason>. Next: <what happens now>.
```

```markdown
# Open questions - <Project name>

1. <question> (asked <who>, <date>)
```

## Daily/YYYY-MM-DD.md

```markdown
---
type: daily
date: YYYY-MM-DD
---
# YYYY-MM-DD

- (claude) 09:12 | Decided: <what>, in [[1 Projects/<slug>/decisions]].
- (chatgpt) 13:40 | Shipped: <what>, <link>.
- (claude) 17:05 | Needs <name>: <question>.
```

Target: under 3 KB per day. Insights, not raw data.

---

## 3 Resources/status-vocabulary.md

```markdown
# The status vocabulary

| Value | Means |
|---|---|
| `active` | Live and current. The default. |
| `draft` | Written but not confirmed. Claims in it are not yet trusted. |
| `living` | Auto-generated or continuously rewritten. Never hand-edit. |
| `paused` | Deliberately stopped, expected to resume. |
| `shipped` | Done and delivered. Kept as proof or reference. |
| `archived` | Retired. History, not current truth. |
| `abandoned` | Stopped and not coming back. |
| `unknown` | Not established yet. Should not survive a weekly cleanup. |

One of the eight in `status:`. Anything else goes in `status_note:` (free text).
```

Why: without a fixed list you end up with 45 free-text values and cannot ask
"show me everything paused".

## 3 Resources/confidence-and-claims.md

```markdown
# Say what you know, mark what you guessed

Every claim written into this vault that is not first-hand from <name> carries
its confidence:
- **confirmed** - <name> said it, or it was directly observed in a file
- **MEDIUM** - a real record supports it, but it is unverified
- **LOW** - inference. Say so, and say what would settle it.

A vault that guesses confidently is worse than one with gaps: the guess gets
read back as truth months later. Corrections stay visible ("Corrected <date>:
was X, now Y"); the record of having been wrong is what makes the rest
trustworthy.
```

## 3 Resources/storage-and-naming.md (short version)

```markdown
# Where files live, and what they are called

| Tier | Lives in | Holds |
|---|---|---|
| Brain | `~/Vaults/OS/` | Markdown only. Never binaries, media or zips. |
| Workbench | `~/Work/<venture>/<project>/` | Code and files for work happening now. |
| Warehouse | <Drive>/Assets/<brand>/<kind>/YYYY-MM/ | Video, images, PDFs, exports, contracts, invoices. |
| Documents | <iCloud or similar> | Personal IDs and family paperwork only. |

Naming: `YYYY-MM-DD_brand_what-it-is_version.ext` for every new file.
Pointer rule: when something heavy moves off the Mac, the project keeps a note
saying where it went, with the clickable link.
Never put a Finder alias or symlink in the vault: both break silently.
```

## 3 Resources/tasks.md (written for list-based tools; adapt to yours)

Trello, Todoist, Linear, Notion and Asana can all hold these six lists (or
statuses). Apple Reminders: use six lists with the same names. No task tool
yet: suggest picking one before Phase 4 (Trello's free plan is enough) rather
than keeping tasks in the vault.

```markdown
# Tasks - how the OS and the task tool divide the work

The OS holds what <name> knows and decided. <Task tool> holds what someone has
to do. GitHub issues hold code work. One task lives in one place.

Lists are status only: Inbox, Next, Doing, Waiting On, Later, Done. Time lives
only in the due date, and only a real deadline gets one.

- New task (any AI): create it in Inbox. Title = the action. Never Next or
  Later; prioritising is <name>'s.
- Close the loop: starting work that has a card, move it to Doing; finished,
  move it to Done in the same turn with a one-line comment and a link.
  Blocked: Waiting On, and say who.
- No task connector in this tool: write "Tasks: done <link>" or
  "Tasks: new - <task>" in today's Daily note; the next session applies it.
- Code work stays a GitHub issue. It also gets a card only with a deadline, a
  waiting client, or something that needs <name>.
- No TODO.md and no checkboxes standing in for tasks anywhere in the OS.
```

Why: when only the human could close cards, 46 cards piled up in Inbox and 2
ever reached Done. Letting any AI that finishes the work close the card fixed it.

## 3 Resources/playbooks/new-project.md (the checklist)

```markdown
# Starting anything new - every agent, every tool

0. Reach the OS. Read AGENTS.md and MAP.md.
1. Prove it is new: read _REGISTER.md, search the vault for the name and its
   synonyms, search the task tool, check GitHub. Exists? Stop, say where it
   lives, and continue THAT work.
2. Classify: client, job, venture, internal tool, small job or idea. Small job
   or idea: no folder, skip to step 7.
3. Ask once, in one message, only what the vault does not answer: what it is and
   for whom, what done looks like, files expected, code repo.
4. Create `1 Projects/<slug>/` with README.md, decisions.md, open-questions.md
   from the templates. Slug: lowercase, hyphenated, English.
5. Register it: new row in _REGISTER.md.
6. Outside the vault, only what step 3 said is needed (Drive folder, repo), each
   with its row in the README "Where things live" table. A repo gets a pointer
   back to the vault folder in its CLAUDE.md / AGENTS.md.
7. Tasks to the task inbox. Never a task list in the vault.
8. If it gets its own workspace (Claude Project, ChatGPT Project, Gem), paste the
   project link block into its instructions.
9. One Daily line linking the README.
10. Verify in a fresh session: "What is this project and where does it live in
    my OS?" It must name the README path.
```

Why: the register exists because people rebuild things they already started.

## Context/ skeletons

Six files, written by the interview (`interview.md`). Each starts as
`status: draft` and becomes `active` with a `reviewed:` date once every `(?)`
is gone.

| File | Holds |
|---|---|
| `me.md` | Identity, in your own words, career and craft, how and when you work, what you believe that others in your field do not, personal context you want AIs to know |
| `business.md` | Each venture: what it is, how the money works, who does what |
| `brand.md` | Positioning, voice rules that proved out, banned words, what the brand is deliberately not, the quality bar, where brand assets live, one section per voice (you, a partner) |
| `icp.md` | Each customer type, where leads really come from, qualification, objections and the answer that works, the anti-customer, the perfect-fit lead in your words |
| `strategy.md` | The plan, standing constraints, the open questions that decide the next 12 months, the three-year picture, the 12-month number |
| `infrastructure.md` | Source of truth per thing (which system holds customers, payments, tasks), tooling, what is retired, direction of travel |

Frontmatter for each:

```yaml
---
type: context
date: YYYY-MM-DD
status: draft        # becomes active after the interview closes every (?)
reviewed: YYYY-MM-DD # set only by you
tags: [context]
---
```
