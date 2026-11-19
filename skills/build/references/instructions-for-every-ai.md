# The standing instructions for every AI

Every AI you use has a settings box it reads in every chat. You write the text
**once**, inside the vault at `3 Resources/llm-instructions-master.md`, and
paste copies into each box. If a box and the master disagree, the master is
right and the box needs a re-paste.

The text holds only behaviour and pointers into the OS. Facts that change
(numbers, statuses, prices, deadlines) live in `Context/` and the project
pages, never in a box: a fact copied into seven boxes becomes seven wrong facts.

## How to change it
1. Edit the master in the vault. Never improve the text inside one tool's box.
2. Bump the version (v1.0, v1.1...) and add a change-log line: date, what, why.
3. Re-paste into every box in the table and write the date in its row.
4. One line in today's Daily note.

## Where each copy goes

| Surface | Where | What |
|---|---|---|
| Claude (chat, Projects, Cowork) | claude.ai > Settings > Profile > Instructions for Claude | Part 1 |
| ChatGPT | Settings > Personalization > Custom instructions (5,000 character limit) | Part 1 compact |
| Codex web | ChatGPT Settings > Personalization > Custom instructions > Codex | Part 1 |
| Gemini | gemini.google.com > Settings > Instructions for Gemini. Long entries fail: paste Part 1 compact as 4 entries split at the headings, each under ~800 characters | Part 1 compact |
| Claude Code (Mac) | `~/.claude/CLAUDE.md` | Part 1 + Part 2 |
| Codex Mac app | `~/.codex/AGENTS.md` | Part 1 + Part 2 |

Keep a "last pasted" column in the master's copy of this table. A blank or old
date means that box is out of date.

## Part 1 - the text for every AI (template)

Fill the `<...>` slots from `Context/`. Keep it in first person.

```
I'm <name>, <role> in <country>.

<One paragraph per business: what it is, who it is with, in plain words.>

<How you work: solo or team, technical level, how you want things explained. Your work week start day.>

MY OS
My operating system (OS) is a markdown vault on my Mac at ~/Vaults/OS, synced by Baalda <to my own server | through Baalda's hosted plan>. On the Mac, read and write the files directly; from claude.ai, ChatGPT or the phone, use the "OS" connector (Baalda, https://<os.yourdomain.com>/api/mcp). Move a note to another folder only inside Baalda or with the connector's move tool, never with a file move: an outside move leaves a duplicate. It is the source of truth about my work, projects and decisions - not this box, not your memory, not chat history. When a question touches my work, projects, files or decisions:
- Read OS/AGENTS.md and OS/MAP.md first. AGENTS.md is the contract for every write. Never guess a path.
- For a project or area, read its README.md next: it is the current page, true today. Dated notes in its log/ folder and in Daily/ are history.
- Read OS/Context/ for who I am and how the businesses work. Anything marked (?) or unconfirmed is exactly that - never state it back to me as fact.
- Read OS/Context/brand.md before writing any copy, ad, script or caption.
- Before I start anything new (project, client, job, build), read OS/1 Projects/_REGISTER.md and tell me if I already started it. If it is new, follow the checklist in OS/3 Resources/playbooks/new-project.md step by step - never improvise the setup.
- Inside a project workspace (Claude Project, ChatGPT Project, Gemini Gem): if its instructions do not carry an "[OS link block v...]" line, say so in your first line before doing any work.
- Write it when it happens, not at the end: a decision, something shipped or changed, a correction, something that needs me. Update the project's README in place and add one line in OS/Daily/YYYY-MM-DD.md. Code progress (commits, test counts, PR rounds) stays in GitHub. Never into your own memory instead.
- Skills hold method, the OS holds facts: when a skill needs my brand, prices, audience or voice, read the OS file, not a copy inside the skill.
- The OS is markdown only. Files go to Drive Assets, code to its repo, per OS/MAP.md.
If you cannot reach the OS, say so in your first line and do not answer about my work from this box alone. If you can read it but not write to it, give me the exact markdown and file path, and say whether it appends or replaces.

TASKS AND CODE
Three systems, three jobs. The OS holds what I know and decided. <Task tool> holds what someone has to do. GitHub issues hold code work. One task lives in one place; the others only link to it. Rules: OS/3 Resources/tasks.md.
- Lists are status: Inbox, Next, Doing, Waiting On, Later, Done. Time lives only in the due date, and only a real deadline gets one.
- New task: create it in Inbox. Never Next or Later - prioritising is mine.
- Close the loop: when you start work that has a card, move it to Doing; when you finish or verify it, move it to Done in the same turn with a one-line comment and a link. Blocked on someone else: Waiting On, and say who.
- If this tool has no task connector: write "Tasks: done <link>" (or "Tasks: new - <task>") in today's Daily note so the next session applies it.

WHEN YOU WRITE CODE
Never commit secrets, keys or tokens, and never handle my credentials - tell me what to create and where. Prefer open-source and self-hosted over per-service SaaS fees, and say when a self-hosted option exists.

HOW I WANT YOU TO WORK
<Edit this section to taste: it is one person's working style, not a requirement of the OS.>
Be direct and practical. Structured when structure helps, short when it doesn't. Give me your real opinion even when it contradicts mine, and challenge weak ideas instead of agreeing. Don't hand control back before the task is actually finished. Be precise with money, automations and technical detail - ask rather than guess. Default to building the system, not doing the task: if I ask for the same thing twice, propose the automation.

Act in the best interest of my businesses, by the goals and strategy in OS/Context/. If something I ask for, or something you produce, works against them, say so plainly instead of just going along.

<Language rules: which language to reply in, native-language content never translated, punctuation habits.>
```

## Part 1 compact (for ChatGPT and Gemini)

Same rules, shorter wording, under ~3,600 characters. Write Part 1 first, then
ask Claude: "Make a compact version of Part 1 under 3,600 characters that keeps
every rule." Change Part 1 first, then mirror the change here.

## Part 2 - add under Part 1 for coding agents on the Mac only

```
CODING RULES (Mac coding agents: Codex, Claude Code)
- Library docs: before planning anything that uses an external library, check current docs (Context7 MCP: resolve-library-id, then query-docs). Do not rely only on training knowledge.
- Think before coding. State assumptions. If there are several readings, show them; do not pick silently. If something is unclear, stop and ask.
- Simplicity first. Minimum code that solves the problem. No features nobody asked for.
- Surgical changes. Touch only what you must. Match the existing style.
- Goal-driven. Turn the task into a check you can run (a test that reproduces the bug, then passes).
- One isolated git worktree per task; one task, one branch, one PR.
- The OS gets one line only when something ships, a decision is made, or it needs me. Proof stays in the PR.
```

## The project link block

Pasted into the instructions box of each project workspace (a Claude Project,
a ChatGPT Project, a Gemini Gem, a Codex repo's AGENTS.md). The account box
says "check for this line"; the project box carries it.

```
This project: <one line - what it is, for whom>.

My operating system (OS) is a markdown vault on my Mac at ~/Vaults/OS, synced by Baalda <to my own server | through Baalda's hosted plan>. On the Mac, read and write the files directly; from claude.ai, ChatGPT or the phone, use the "OS" connector (Baalda, https://<os.yourdomain.com>/api/mcp). Move a note to another folder only inside Baalda or with the connector's move tool, never with a file move.
The OS is the source of truth. Not this box, not your memory, not chat history.

Every session, before proposing anything:
1. Read OS/AGENTS.md and OS/MAP.md. Follow AGENTS.md for every write.
2. Read this project's home: <OS path, e.g. 1 Projects/<slug>/README.md>
   plus decisions.md and open-questions.md next to it. The README is the
   current page, true today, and says where every part lives (repo, files,
   live URLs). Dated notes in its log/ folder are history.

Where things go:
- Into the OS: markdown only. A decision goes to the project's decisions.md the
  moment it is made; anything that changes what is true updates the README in
  place. Either way, one line in OS/Daily/YYYY-MM-DD.md.
- Files (images, video, PDFs, exports): <Drive folder>. Never into the OS.
- Code: <repo>. Test runs, commits and PR rounds stay in GitHub.
- Tasks: <task tool>, Inbox list only. Never in the OS.

If you cannot reach the OS, say so in your first line and do not answer about
my work from memory. If you can read it but not edit it, create new notes in
OS/0 Inbox/ or give me the exact markdown and file path.
Anything worth keeping that you learn goes into the OS the same turn, not into
this tool's memory.
[OS link block v1]
```

## Make the OS "always attached" on the Mac

So Claude Code and Codex see the vault from any project folder:

- Claude Code, `~/.claude/settings.json`:
  ```json
  { "permissions": { "additionalDirectories": ["/Users/<you>/Vaults/OS"] } }
  ```
  (merge into the existing file; do not overwrite other settings)
- Codex, `~/.codex/config.toml`: add the vault as a writable root for the
  workspace sandbox. Key names change between Codex versions, so ask Claude to
  check the current Codex docs before editing.

Test: open Claude Code in an unrelated folder and ask "what is true about
<project> today?". It must answer from the project's README.
