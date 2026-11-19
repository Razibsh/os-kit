---
name: build
description: Build a personal AI operating system from zero with the user - a markdown second brain that Claude, ChatGPT, Gemini and Codex all read and update, synced by Baalda on the user's own server or Baalda's hosted plan, kept clean by small automations. Covers the vault, a voice interview, the server (with or without Coolify), connectors for every AI, skills, guarded backups and cloud routines. Run only when the user asks to build or set up their OS with this kit.
argument-hint: "[optional: where your existing notes are]"
disable-model-invocation: true
---

# Build your OS

You are going to build the user's personal AI operating system ("the OS") with
them, from zero, following this kit. The kit is a tested recipe from someone who
built and ran it for months; follow it rather than improvising. The reference
files sit in `references/` next to this file.

## Before anything else

Read these in one batch: `references/how-it-works.md`,
`references/vault-templates.md`, `references/interview.md`,
`references/server-from-zero.md`, `references/baalda-and-connectors.md`,
`references/instructions-for-every-ai.md`, `references/skills.md`,
`references/automation.md`.

## Phase 0 - Check where we are (2 minutes)

1. **Terminal?** If you cannot run shell commands in this session (you are in
   claude.ai chat, not Claude Code), say so plainly: the vault and the
   interview can happen here, but the server and the Mac automation need Claude
   Code. Tell them to install the Claude desktop app (https://claude.ai/download),
   open its **Code** tab, and paste the same kit link there. Offer to start the
   interview here meanwhile, handing them each file's text to save.
2. **System.** Run `uname -s` and `sw_vers` (Mac) to learn the OS. Not macOS:
   tell them up front which parts work as is and which you will port (table at
   the end of `references/server-from-zero.md`).
3. **Existing OS?** Check whether `~/Vaults/OS/AGENTS.md` (or the folder they
   name) already exists. If it does, stop and ask: this kit builds a new OS. For
   an existing one, offer a read-only review against the kit instead, and change
   nothing without their numbered approval.
4. Then tell them in 5 lines what you are about to build, and ask the setup
   questions below in ONE message.

## How to work with the user

- Assume they are not technical. Explain the why in plain words. No jargon
  without a one-line explanation.
- One phase at a time. At the start of each phase, say what it does and how
  much of their time it takes. At the end, run its check and show the result.
- Before anything hard to undo or public (a DNS record, a deploy, opening a
  port, deleting or moving files, scheduling a routine, publishing a repo), stop
  and get their explicit yes.
- **Never type, ask for, or store their passwords, API keys, tokens or payment
  details** in chat or in any file. Tell them exactly where to paste each one
  themselves (a dashboard, their password manager, macOS Keychain).
- Once the vault exists, log each decision or finished step as one line in
  `Daily/<today>.md`, as the kit describes.
- If a step no longer matches what you see (a menu moved, a release changed),
  check the current docs, tell them what changed, and adapt. Never guess a
  setting.

## Setup questions (one message)

1. Your first name, and what you do (one line per business or job).
2. Do you already have notes somewhere (Obsidian, Notion export, Apple Notes,
   Google Docs, a folder)? Where?
3. Server: (a) I have a server with Coolify, (b) I have a server without
   Coolify, (c) I have no server, (d) I don't know what this means. For (a)
   or (b): do you have a domain, and where is its DNS managed?
4. Which AIs do you use? (Claude, Claude Code, ChatGPT, Codex, Gemini)
5. Where do your tasks live today (Trello, Todoist, Linear, Notion, Apple
   Reminders, nowhere)?
6. Where should heavy files go (Google Drive, Dropbox, iCloud, OneDrive)?
7. Do you have a GitHub account? (Needed for the private hourly backup. If not,
   you will help them create one; it is free.)
8. Your time zone and the day your work week starts.
9. Which language(s) should the AIs answer in?

For (c) or (d), explain the two options in `references/server-from-zero.md`
(Baalda's hosted plan, or renting a small server) and let them choose before
Phase 3.

## Phase 1 - The vault skeleton (about 20 min, 5 of them theirs)

- Create `~/Vaults/OS` with the folder tree in `references/vault-templates.md`.
  Not in `~/Documents`, iCloud, Google Drive, Dropbox or OneDrive: one sync
  engine only, and that will be Baalda.
- Write AGENTS.md, CLAUDE.md, MAP.md, START-HERE.md, `1 Projects/_REGISTER.md`
  and the `3 Resources/` rule files from the templates, filled with what you know
  so far. Unknowns say EMPTY ON PURPOSE.
- Add to AGENTS.md from day one the weekly-cleanup exception from
  `references/automation.md` (up to 10 mechanical fixes per run). Without it,
  the cleanup later refuses to fix anything.
- Existing notes: back them up first (`tar` to their home folder), propose
  where each kind of note goes in the new tree, and wait for their yes before
  copying. Copy, never move, the originals.
- **Check:** show the tree and the line count of AGENTS.md (under 200).

## Phase 2 - The interview (60-90 min of theirs, can be split)

- Run `references/interview.md` as written: one section at a time, they answer
  by voice, you write each Context file right after its section, plus register
  rows and project folders.
- Close the loop: settle every `(?)`, then `status: active` and
  `reviewed: <today>` on each Context file.
- **Check:** zero `(?)` left in `Context/`, every Context file active with a
  reviewed date, every active project has a README.

## Phase 3 - Server and Baalda (one evening, about 30 min theirs)

- No server or no Coolify: `references/server-from-zero.md` first.
- Then `references/baalda-and-connectors.md` for their path:
  - Hosted plan: the Mac side only.
  - Self-hosted: their own private copy of Baalda at a pinned release, the
    compose changes, the invite-only patch (apply it and show them the diff),
    DNS (their yes first), deploy (their yes first). They create their account
    as the first user. Then prove from outside that a made-up sign-up is refused.
    Nightly database dump, and a way to copy it off the server.
- Mac: the rehearsal vault first (show them the move-creates-a-duplicate test),
  then open `~/Vaults/OS` in Baalda, sync on, Freeze vault root on.
- Make the vault a standing folder for Claude Code (and Codex if they use it).
- **Check:** the note count in Baalda matches the folder; self-hosted: `/health`
  answers and a made-up sign-up is refused from outside.

## Phase 4 - Connect every AI (about 20 min theirs)

- Walk them through the connector for each AI they named, called **OS**
  everywhere.
- Write `3 Resources/llm-instructions-master.md` from
  `references/instructions-for-every-ai.md`, filled from `Context/`, plus the
  compact version. Write `~/.claude/CLAUDE.md` (Part 1 + Part 2) yourself after
  showing a diff of any existing file. For each web AI, give them the exact text
  and the exact place to paste it, one AI at a time.
- **Check:** in each AI, in a new chat: "Use the OS connector: read AGENTS.md and
  tell me its first heading." Then the phone test with the laptop closed: add a
  line to today's Daily note, and confirm it reached the Mac.

## Phase 5 - Skills (about 30 min)

- Help them install Ben AI's free skills from the links in
  `references/skills.md` (never bundle or copy them: they are his work).
- Run the optimizer once in "save everything to plan" mode on the new vault and
  show the score and the plan. Change nothing without their numbered approval.
- Once the vault exists, remove or disable the setup skill (its reset option is
  dangerous on an existing vault).
- If they have their own skills: find business facts inside them, merge what
  `Context/` lacks, and replace the copies with pointers to the OS.
- **Check:** a skill run in a fresh session reflects a one-line change they make
  to `Context/brand.md`.

## Phase 6 - Automation (1-2 hours of yours, about 15 min theirs)

- Mac: a private backup repo, `vault-backup.sh`, `daily-capture.sh`,
  `os-doctor.sh` and their launchd jobs from `references/automation.md`, in
  `~/os-scripts`. Build the OSJobs launcher only if the readability probe fails
  under launchd. Not a Mac: port them, as agreed in Phase 0.
- Test the backup guard on a temporary copy: 20 notes removed must be refused;
  one emptied note must be refused. Prove each job through
  `launchctl kickstart`, not by running it by hand.
- Cloud: write the two routine notes into `3 Resources/`, set up the private
  `os-routines` repo with the optimizer and house rules, then create the three
  routines (their yes before scheduling). Convert their local times to UTC cron
  and tell them the two dates a year to re-check them.
- Run each routine once by hand and show what it wrote.
- **Check:** `os-health.md` shows every job green with a real last-run time.

## Phase 7 - Hand over

- Update START-HERE.md and MAP.md so they describe exactly what now runs.
- Give them a one-page summary: what runs when, what they do each week (answer
  the approve list, about 5 minutes), and the three things that would break it
  (moving notes outside Baalda, a second sync engine on the folder, binaries in
  the vault).
- Log the build in today's Daily note and in `Notes/decisions/`.
- Ask what went off script in this run, and if they are willing, suggest they
  tell the kit's author (an issue on the kit's GitHub repo) so it gets fixed for
  the next person.
