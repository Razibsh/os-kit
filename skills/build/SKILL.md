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

## Phase 0 - Check where we are (5 minutes)

Do these checks before reading the reference files.

1. **Terminal?** If you cannot run shell commands in this session (you are in
   claude.ai chat, not Claude Code), say so plainly in your first reply: the
   vault and the interview can happen here, but the server and the Mac
   automation need Claude Code. Tell them to install the Claude desktop app
   (https://claude.ai/download), open its **Code** tab, and paste the same kit
   link there. Offer to start the interview here meanwhile, handing them each
   file's text to save.
2. **System.** Run `uname -s` (and `sw_vers` on a Mac). Not macOS: tell them up
   front which parts work as is and which you will port (table at the end of
   `references/server-from-zero.md`).
3. **Existing OS?** Check whether `~/Vaults/OS/AGENTS.md` exists. If it does,
   stop: this kit builds a new OS, and running it would overwrite theirs. Offer
   three choices: (1) a read-only review of their OS against the kit (the
   checklist at the end of this file), (2) a second, separate OS in another
   folder, (3) something else. Change nothing until they choose.
4. **Tools on a fresh Mac.** Check `xcode-select -p`, `git --version`,
   `gh --version`. Explain what each is in one line.
   - Missing Command Line Tools (git, clang and python3 come with them): the
     person runs `xcode-select --install` and clicks Install in the window that
     opens.
   - Missing `gh` (GitHub's command-line tool): with Homebrew, `brew install gh`;
     without it, the installer from https://cli.github.com.
   - Then the person signs in themselves, in the Terminal app, because it asks
     questions you cannot answer from here: `gh auth login` (choose GitHub.com,
     HTTPS, log in with a web browser). Check with `gh auth status`.
   - General rule: any command that prompts for input (passwords, passphrases,
     sign-ins, `sudo`) is run by the person in the Terminal app, never by you.
5. Now read the reference files in one batch: `references/how-it-works.md`,
   `references/vault-templates.md`, `references/interview.md`,
   `references/server-from-zero.md`, `references/baalda-and-connectors.md`,
   `references/instructions-for-every-ai.md`, `references/skills.md`,
   `references/automation.md`.
6. Tell them in 5 lines what you are about to build, and ask the setup
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
   Google Docs, a folder)? Where? (If one of them is already an AI-maintained
   vault with an AGENTS.md, say so: you will offer the review instead.)
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
- Keep the weekly-cleanup exception that the AGENTS.md template carries (up
  to 10 mechanical fixes per run). Without it, the cleanup later refuses to
  fix anything.
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
- **Check:** every top-level folder shows in Baalda and random notes open with
  their content; self-hosted: `/health` answers and a made-up sign-up is
  refused from outside. (Phase 4 checks the exact note count through the
  connector.)

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
  `references/skills.md` (never bundle or copy them: they are his work). The
  optimizer is the one the weekly cleanup needs; the operator is optional (the
  kit already has an adapted daily update); Ben's setup skill is not needed.
- Run the optimizer once in "save everything to plan" mode on the new vault and
  show the score and the plan. Change nothing without their numbered approval.
- If they installed **Ben's** `/second-brain-setup` skill (not this kit),
  remove or disable it once the vault exists: it can trigger on everyday words
  and offers a full reset of an existing vault.
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
- **Check:** `2 Areas/personal/os-health.md` shows the backup and the capture
  green with a real last-run time, and each of the three routines has run once
  by hand and written what it should (a Daily line or report, a cleanup note,
  a brief).

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

## Reviewing an OS that already exists (read-only)

Use this when Phase 0 found an existing vault and they chose the review. Read,
compare, and return one numbered list of gaps, smallest change first. Change
nothing until they approve items by number.

1. Rules: is there an `AGENTS.md` under 200 lines, a `CLAUDE.md` that only
   imports it, a `MAP.md`? Does AGENTS.md have the two kinds of notes, one fact
   one home, the move rule for Baalda, the weekly-cleanup exception?
2. Lifecycle: do projects have one current README each, with dated notes in
   `log/`? Any README over ~150 lines of logs? Daily notes over 3 KB?
3. Context: which `Context/` files exist, their status, `reviewed:` dates, and
   open `(?)`.
4. Sync and safety: one sync engine only? `.context/` excluded from any backup?
   Is there a guarded backup (refuses mass deletions)? When did it last run?
5. Jobs: which scheduled jobs exist, do they write heartbeats, are they green?
6. Connectors: which AIs reach the OS, and do their standing instructions point
   at it?
7. Skills: do any skills carry their own copies of business facts?

## Things that change: re-check before you rely on them

These were true when the kit was written (Nov 2026). Before telling the person
to click something on this list, check the current docs or ask them what they
see, and say so if it differs:
- Baalda menus ("Server settings", "Vault settings > MCP", "Freeze vault
  root", "Make local only"), whether the hosted plan is free and exposes the
  connector, and the latest release tag.
- The connector screens in claude.ai, ChatGPT and Gemini (and that Gemini names
  it "Context" by default).
- Claude cloud routines: where they are managed (claude.ai/code/routines at the
  time), how a routine is given access to private GitHub repos (connect GitHub
  for Claude Code on the web and allow those repos), and the folder repos are
  cloned into.
- Coolify's install command and ports, and Codex's config keys.
