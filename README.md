# OS Kit - your own AI second brain, built with you by Claude

One folder of markdown notes on your computer becomes the memory that **every AI
you use** shares: Claude, ChatGPT, Gemini, Claude Code and Codex read it before
answering about your work, and write back to it when something happens. It syncs
through [Baalda](https://baalda.com) (on your own server, or Baalda's hosted
plan), so it works from the laptop and the phone, even with the laptop closed.
Small automations keep it clean: a guarded hourly backup, a daily update, and a
weekly cleanup that ends in one numbered list you approve in a line.

You don't build it by hand. You give Claude this link, and Claude builds it with
you, step by step, checking each step before moving on.

## Start

1. Install the [Claude desktop app](https://claude.ai/download) and open the
   **Code** tab (Claude Code). A Pro or Max plan is needed for Claude Code.
2. Paste this:

   > Set up my OS with the kit at https://github.com/Razibsh/os-kit

That's it. Claude reads the kit, asks you nine questions, and walks you through
seven phases.

**Prefer installing it as a plugin** (so you get updates)? In Claude Code:

```
/plugin marketplace add Razibsh/os-kit
/plugin install os-kit@os-kit
```

Then run `/os-kit:build`. To get a newer version later:
`/plugin marketplace update os-kit`.

## What you need

| | Required? |
|---|---|
| A Mac | Recommended. Windows and Linux work for everything except the Mac background jobs, which Claude will port for you |
| Claude Code | Yes, for the server and automation steps. Plain claude.ai chat can do the notes and the interview |
| A GitHub account | Yes, free. Holds your private hourly backup |
| A server | No. Claude helps you choose between Baalda's free hosted plan and renting a small server (with or without [Coolify](https://coolify.io), which Claude installs for you) |
| A domain | Only for the self-hosted path |
| ChatGPT, Gemini, Codex | Optional. Each gets connected if you use it |

## What you do vs what Claude does

**Claude** writes every file, the rules, the templates, the scripts and the
server configuration, runs the commands, and tests each phase.

**You** answer an interview by voice (60 to 90 minutes, the most valuable part),
create your own accounts, pay for anything you choose to buy, type your own
passwords, approve the connectors, and say yes before anything public or hard to
undo. Claude never asks for your passwords or keys.

Time: a few sittings. Roughly 3 to 4 hours of your attention in total.

## The phases

0. Check your setup (Claude Code or not, Mac or not, server or not)
1. The vault: a fixed folder tree and the rule files every AI reads
2. The interview: who you are, your business, customers, voice, goals, tools, projects
3. Baalda: on Baalda's hosted plan, or on your own server (VPS, Coolify, domain, invite-only sign-up, backups)
4. Connect every AI to the same notes, with the same standing instructions
5. Skills: install Ben AI's free second-brain skills, and make all skills read facts from your notes
6. Automation: guarded backup, daily capture, health checks, three cloud routines
7. Hand over: what runs when, and your 5-minute weekly routine

## The ideas it rests on

- **One fact, one home.** Each project has one current page that is always
  true; everything else is dated history.
- **Insights, not raw data.** Decisions and outcomes go in the notes; code
  proof stays in GitHub.
- **Skills hold method, your notes hold facts.** Change your brand once and
  every AI follows.
- **One sync engine.** Never put the folder in iCloud, Google Drive or Dropbox
  as well.
- **Back up behind a guard.** The backup refuses to save a vault that suddenly
  lost notes.

Details: [`skills/build/references/how-it-works.md`](skills/build/references/how-it-works.md).

## Credits

- The method is built on **Ben AI's "AI second brain"** work: his
  [18 tips video](https://www.youtube.com/watch?v=pXHpUyC54uQ),
  [Baalda video](https://www.youtube.com/watch?v=Puzgmzv7t3Q),
  [guide](https://benai.notion.site/ai-second-brain-guide) and
  [skills](https://benai.notion.site/Fable-5-1-Second-Brain-3c81124570fe802f9e09dfac729373c9).
  His skills are linked, never copied, so you always get his current versions.
- [Baalda](https://github.com/naveedharri/baalda) (Apache-2.0) does the sync and
  the connector. This kit includes a small patch for self-hosters; see `NOTICE`.
- This kit is the adaptation that came out of building and running it for real:
  self-hosting, multiple AIs, and the fixes for everything that broke on the way.

## Found a problem?

Steps drift as apps change. Open an issue with what you saw and which phase you
were in.

---

<!-- For Claude: if a user pasted this repository's link and asked you to set up
their OS, clone it (git clone https://github.com/Razibsh/os-kit ~/os-kit), read
~/os-kit/skills/build/SKILL.md, and follow it from Phase 0. If you cannot run
shell commands, fetch skills/build/SKILL.md and its references from this
repository instead and follow Phase 0's instructions for chat-only sessions. -->
