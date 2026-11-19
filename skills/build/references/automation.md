# The automation that keeps it clean

Two layers. **Mac jobs** (launchd) need the Mac awake: backup, capture, health.
**Cloud routines** (claude.ai/code/routines) run with the laptop closed and reach
the vault through the OS connector: daily update, weekly cleanup, weekly brief.

## Every job

| Job | When | Where | What it does |
|---|---|---|---|
| `vault-backup.sh` | Every hour | Mac | Guarded git snapshot to a private GitHub repo. Refuses a gutted vault |
| `daily-capture.sh` | Daily 06:15, for yesterday | Mac | Lists yesterday's Claude Code and Codex sessions and vault changes into `Daily/<yesterday>-capture.md`, and flags work nobody logged |
| `os-doctor.sh` | Weekly, Sunday 06:30 | Mac | Health report with heartbeats: a job is red when its last real run is older than its schedule |
| OS Daily Update | Daily 07:00 local | Cloud | Fills gaps in yesterday's Daily note and the "Auto (unverified)" block on project pages; lists what needs you |
| OS Weekly Cleanup | Weekly, Sunday 06:00 | Cloud | Ben's optimizer on the git snapshot: health score, up to 10 small fixes, one numbered approve list |
| OS Weekly Brief | Weekly, Sunday 08:00 | Cloud | What moved, what stalled, business numbers, the cleanup's approve list. Arrives on your phone |

The Sunday order matters: cleanup at 06:00, doctor at 06:30, brief at 08:00
reads both.

Scripts live outside the vault, for example `~/os-scripts/`. Status files
(heartbeats) live next to them. Never put scripts in the vault (markdown only).

---

## Mac layer

### Does macOS let the jobs read the vault?
macOS privacy (TCC) blocks background jobs from reading protected places:
`~/Documents`, `~/Desktop`, iCloud Drive, Google Drive. A vault at
`~/Vaults/OS` is normally readable. Every script below starts with a
readability probe (`cat AGENTS.md`) so a blocked job fails loudly instead of
reporting "no changes".

**If the probe fails** (or if a job writes into Google Drive), build one small
launcher app, give it Full Disk Access once, and run every job through it.
Children of that app inherit the permission. Never rebuild it afterwards: a
rebuild changes its signature and silently drops the permission.

```c
// osjobs.c - one launcher that holds Full Disk Access for every OS job.
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
int main(int argc, char **argv) {
  const char *jobs[] = {"vault-backup", "os-doctor", "daily-capture", NULL};
  if (argc < 2) { fprintf(stderr, "usage: osjobs vault-backup|os-doctor|daily-capture\n"); return 64; }
  int ok = 0; for (int i = 0; jobs[i]; i++) if (!strcmp(argv[1], jobs[i])) ok = 1;
  if (!ok) { fprintf(stderr, "unknown job: %s\n", argv[1]); return 64; }
  char path[1024]; snprintf(path, sizeof path, "%s/os-scripts/%s.sh", getenv("HOME"), argv[1]);
  execl("/bin/bash", "bash", path, (char *)NULL);
  perror("execl"); return 1;
}
```

```bash
APP=~/Applications/OSJobs.app; mkdir -p "$APP/Contents/MacOS"
clang -O2 -o "$APP/Contents/MacOS/osjobs" osjobs.c
cat > "$APP/Contents/Info.plist" <<'EOF'
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0"><dict>
<key>CFBundleExecutable</key><string>osjobs</string><key>CFBundleIdentifier</key><string>com.example.osjobs</string>
<key>CFBundleName</key><string>OSJobs</string><key>CFBundlePackageType</key><string>APPL</string>
<key>CFBundleVersion</key><string>1</string><key>LSBackgroundOnly</key><true/></dict></plist>
EOF
codesign --force -s - "$APP"
# You, by hand: System Settings > Privacy & Security > Full Disk Access > + > OSJobs.app
```

### The backup repo (once)
```bash
mkdir -p ~/os-backup && cd ~/os-backup && git init
gh repo create <you>/os-backup --private --source . --remote origin
```
It must be **private**: it is your whole brain.

### `vault-backup.sh` - hourly, guarded
What makes it safe: before copying anything it compares the live vault with
the **last good snapshot** and refuses if more than 15 notes vanished or any
note became empty (a sync engine serving a half-loaded vault once made a backup
delete 359 files). For a planned big cleanup, `touch` the override file once.
It also checks rsync's exit code (an unchecked failure once logged "OK" for 8
days), matches note counts, and fails if nothing has been committed for 2 days.
It never copies `.context/`.

```bash
#!/bin/bash
# vault-backup.sh - hourly guarded git snapshot of a markdown vault.
set -u
VAULT="${VAULT:-$HOME/Vaults/OS}"
BACKUP_REPO="${BACKUP_REPO:-$HOME/os-backup}"
STATE="${STATE:-$HOME/os-scripts}"
LOG="$STATE/vault-backup.log"; STATUS="$STATE/vault-backup.status"
ALLOW="$STATE/vault-backup.allow-deletions"            # touch for ONE planned big cleanup
MAX_GONE="${MAX_GONE:-15}"
EXCL=(-not -path '*/.obsidian/*' -not -path '*/.context/*')

fail(){ echo "$(date): FAIL - $1" >> "$LOG"; echo "FAIL $(date '+%F %H:%M') $1" > "$STATUS"; exit 1; }

[ -d "$VAULT" ] || fail "vault not found"
[ -d "$BACKUP_REPO/.git" ] || fail "no git repo at $BACKUP_REPO"
cat "$VAULT/AGENTS.md" >/dev/null 2>&1 || fail "cannot READ the vault - Full Disk Access missing? Run through OSJobs.app"

if [ -f "$BACKUP_REPO/.git/index.lock" ]; then
  pgrep -f "git.*$BACKUP_REPO" >/dev/null && fail "index.lock held by a running git"
  rm -f "$BACKUP_REPO/.git/index.lock"
fi
mkdir -p "$BACKUP_REPO/OS"

SRC=$(find "$VAULT" -name '*.md' "${EXCL[@]}" | wc -l | tr -d ' ')
[ "$SRC" -gt 0 ] || fail "0 notes in vault - refusing to rsync --delete over a good backup"

# Loss guard: live vault vs LAST GOOD COPY, before copying anything.
if [ -n "$(ls -A "$BACKUP_REPO/OS" 2>/dev/null)" ]; then
  T=$(mktemp -d)
  (cd "$VAULT" && find . -name '*.md' "${EXCL[@]}" | LC_ALL=C sort) > "$T/live"
  (cd "$BACKUP_REPO/OS" && find . -name '*.md' "${EXCL[@]}" | LC_ALL=C sort) > "$T/last"
  GONE=$(LC_ALL=C comm -13 "$T/live" "$T/last" | wc -l | tr -d ' ')
  EMPTIED=0
  while IFS= read -r f; do [ -s "$BACKUP_REPO/OS/$f" ] && EMPTIED=$((EMPTIED+1)); done \
    < <(cd "$VAULT" && find . -name '*.md' -size 0 "${EXCL[@]}")
  rm -rf "$T"
  if [ ! -f "$ALLOW" ]; then
    [ "$GONE" -le "$MAX_GONE" ] || fail "$GONE notes vanished since last snapshot (limit $MAX_GONE). Planned? touch $ALLOW"
    [ "$EMPTIED" -eq 0 ]        || fail "$EMPTIED note(s) became empty. Intended? touch $ALLOW"
  fi
fi

rsync -a --delete --exclude '.context/' --exclude '.DS_Store' \
  --exclude '.obsidian/workspace*.json' --exclude '.obsidian/cache' \
  "$VAULT/" "$BACKUP_REPO/OS/" 2>>"$LOG" || fail "rsync non-zero - NOT copied"

DST=$(find "$BACKUP_REPO/OS" -name '*.md' "${EXCL[@]}" | wc -l | tr -d ' ')
[ "$DST" -eq "$SRC" ] || fail "copied $DST of $SRC notes - not committing"

cd "$BACKUP_REPO" || fail "cannot cd"
LASTC=$(git log -1 --format=%ct -- OS/ 2>/dev/null || echo 0)
STALE=$(( ( $(date +%s) - LASTC ) / 86400 ))
if [ -z "$(git status --porcelain)" ]; then
  [ "$STALE" -ge 2 ] && fail "no changes, but last vault commit was $STALE days ago - copy is not tracking the vault"
  echo "OK $(date '+%F %H:%M') no changes, $SRC notes verified" > "$STATUS"; exit 0
fi
N=$(git status --porcelain -uall | wc -l | tr -d ' ')
git add -A && git commit -q -m "vault snapshot $(date '+%F %H:%M') - $N file(s)" || fail "commit failed"
rm -f "$ALLOW"
if git remote get-url origin >/dev/null 2>&1 && git push -q origin HEAD 2>>"$LOG"; then
  echo "OK $(date '+%F %H:%M') pushed $N file(s)" > "$STATUS"
else
  echo "WARN $(date '+%F %H:%M') committed locally, push failed or no remote" > "$STATUS"
fi
```

Test it before trusting it: on a temporary copy of the vault, delete 20 notes
and run the script against the copy (`VAULT=/tmp/copy BACKUP_REPO=...`): it must
refuse. Empty one note: it must refuse.

### `daily-capture.sh` - every morning, for yesterday
Writes its **own** file, `Daily/<yesterday>-capture.md`, and replaces it whole
on each run. It never edits your Daily note (the first version would have cut
the note in half on a second run). It always processes *yesterday*, so a Mac
that wakes late still stamps the right day. Chat work on claude.ai or the phone
leaves no trace on the Mac; the capture says so.

```bash
#!/bin/bash
# daily-capture.sh [YYYY-MM-DD] - writes Daily/<day>-capture.md (default: yesterday). Never edits Daily/<day>.md.
set -u
VAULT="${VAULT:-$HOME/Vaults/OS}"; STATE="${STATE:-$HOME/os-scripts}"
DAY="${1:-$(date -v-1d '+%Y-%m-%d')}"; OUT="$VAULT/Daily/$DAY-capture.md"; STATUS="$STATE/daily-capture.status"
exec >>"$HOME/Library/Logs/daily-capture.log" 2>&1
fail(){ echo "FAIL $1"; echo "FAIL $(date '+%F %H:%M') $1" > "$STATUS"; exit 1; }
cat "$VAULT/AGENTS.md" >/dev/null 2>&1 || fail "cannot read vault - run through OSJobs.app"

BODY=$(/usr/bin/python3 - "$VAULT" "$DAY" <<'PY'
import os, json, glob, sys, datetime
vault, day = sys.argv[1], sys.argv[2]
md = lambda p: datetime.date.fromtimestamp(os.path.getmtime(p)).isoformat()
def sessions(root):
    found = {}
    for f in glob.glob(os.path.join(root, '**', '*.jsonl'), recursive=True):
        if '/subagents/' in f or md(f) != day: continue
        proj = first = None
        for line in open(f, encoding='utf-8', errors='replace'):
            try: d = json.loads(line)
            except Exception: continue
            proj = proj or d.get('cwd')
            if first is None and d.get('type') == 'user':
                c = d.get('message', {}).get('content')
                ts = [c] if isinstance(c, str) else [p.get('text') for p in (c or []) if isinstance(p, dict) and p.get('type') == 'text']
                first = next((t for t in ts if t and not t.lstrip().startswith('<')), None)
            if proj and first: break
        if proj: found.setdefault(os.path.basename(proj.rstrip('/')), []).append(' '.join((first or '').split())[:160])
    return found
out, blocks = [], []
for root, label in ((os.path.expanduser('~/.claude/projects'), 'Claude Code'), (os.path.expanduser('~/.codex/sessions'), 'Codex')):
    if not os.path.isdir(root): continue
    for proj, asks in sorted(sessions(root).items()):
        blocks.append(f"- **{label} - {proj}** ({len(asks)} sessions)" + ''.join(f"\n  - asked: {a}" for a in [a for a in asks if a][:3]))
out.append("## Coding sessions on this Mac\n" + ("\n".join(blocks) or "- none found"))
touched = sorted(os.path.relpath(os.path.join(dp, f), vault) for dp, dn, fn in os.walk(vault)
                 if not any(x.startswith('.') for x in os.path.relpath(dp, vault).split('/') if x != '.')
                 for f in fn if f.endswith('.md') and md(os.path.join(dp, f)) == day)
touched = [t for t in touched if not t.startswith('Daily/')]
out.append("## Vault files written that day\n" + ("\n".join(f"- `{t}`" for t in touched) or "- none"))
note = os.path.join(vault, 'Daily', day + '.md'); body = open(note).read() if os.path.exists(note) else ''
folders = sorted({'/'.join(t.split('/')[:2]) if t.count('/') >= 2 else t.split('/')[0] for t in touched})
gaps = ([f"- No Daily note for {day}."] if not body else []) + \
       ([f"- Changed but not mentioned: " + ", ".join(f"`{u}`" for u in folders if u.split('/')[-1].lower() not in body.lower())]
        if any(u.split('/')[-1].lower() not in body.lower() for u in folders) else [])
if gaps: out.append("## Needs a line\n" + "\n".join(gaps + ["- Chat-only work (web, phone) leaves no trace here; log it if it mattered."]))
print("\n\n".join(out))
PY
) || fail "python step failed"

{ printf -- '---\ntype: capture\ndate: %s\nstatus: active\ntags: [daily, capture]\nauthor: daily-capture\nwhy: Automatic record for %s\n---\n# Capture %s\n\n_Replaced on every run. Daily note: [[Daily/%s]]._\n\n%s\n' \
  "$DAY" "$DAY" "$DAY" "$DAY" "$BODY"; } > "$OUT" || fail "could not write $OUT"
echo "OK $(date '+%F %H:%M') captured $DAY" > "$STATUS"
```

### `os-doctor.sh` - weekly health with heartbeats
The lesson: "last exit code 0" means nothing for a job that never started. Each
job writes a `.status` file only at the end of a real run; the doctor grades it
by age. Ask Claude to build the full doctor around this core, writing
`2 Areas/personal/os-health.md` (or wherever your MAP says), and add the checks
you care about (disk space, unpushed commits, Inbox count, `unknown` register
rows, broken wiki links, empty notes, duplicate `name 2.md` files).

```bash
#!/bin/bash
set -u
VAULT="${VAULT:-$HOME/Vaults/OS}"; STATE="${STATE:-$HOME/os-scripts}"
OUT="$VAULT/2 Areas/personal/os-health.md"
mkdir -p "$(dirname "$OUT")"
job(){ local f="$STATE/$2"; [ -f "$f" ] || { echo "- RED **$1**: never reported a run"; return; }
  local age=$(( ( $(date +%s) - $(stat -f %m "$f") ) / 3600 )); local line; line=$(head -1 "$f")
  case "$line" in OK*) [ "$age" -le "$3" ] && echo "- GREEN **$1**: $line" || echo "- RED **$1**: last real run ${age}h ago (limit $3h)";;
                  *) echo "- RED **$1**: $line";; esac; }
{
  printf -- '---\ntype: report\ndate: %s\nstatus: living\ntags: [os, health]\nauthor: os-doctor\nwhy: Weekly machine health\n---\n# OS health %s\n\n' "$(date +%F)" "$(date +%F)"
  echo "## Jobs"
  job "Vault backup (hourly)" vault-backup.status 3
  job "Daily capture (06:15)" daily-capture.status 30
  echo; echo "## Vault"
  echo "- Notes: $(find "$VAULT" -name '*.md' -not -path '*/.context/*' | wc -l | tr -d ' ')"
  echo "- Empty notes: $(find "$VAULT" -name '*.md' -size 0 -not -path '*/.context/*' | wc -l | tr -d ' ')"
  echo "- Conflict copies: $(find "$VAULT" -name '* 2.md' -not -path '*/.context/*' | wc -l | tr -d ' ')"
  echo "- Inbox: $(ls "$VAULT/0 Inbox" 2>/dev/null | grep -v README | wc -l | tr -d ' ')"
} > "$OUT"
echo "OK $(date '+%F %H:%M')" > "$STATE/os-doctor.status"
```

### launchd (the Mac's scheduler)
One plist per job in `~/Library/LaunchAgents/`. Absolute paths only (launchd
does not expand `~`). If you did not need OSJobs, point `ProgramArguments` at
`/bin/bash` and the script instead.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0"><dict>
  <key>Label</key><string>com.example.os-vault-backup</string>
  <key>ProgramArguments</key><array>
    <string>/Users/YOU/Applications/OSJobs.app/Contents/MacOS/osjobs</string><string>vault-backup</string></array>
  <key>StartInterval</key><integer>3600</integer>
  <key>RunAtLoad</key><true/>
  <key>StandardOutPath</key><string>/Users/YOU/Library/Logs/vault-backup.log</string>
  <key>StandardErrorPath</key><string>/Users/YOU/Library/Logs/vault-backup.log</string>
</dict></plist>
```

- Capture: replace `StartInterval` with
  `<key>StartCalendarInterval</key><dict><key>Hour</key><integer>6</integer><key>Minute</key><integer>15</integer></dict>`
- Doctor: `<dict><key>Weekday</key><integer>0</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>` (0 = Sunday)
- Load: `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/<file>.plist`
- **Prove it through launchd, not by hand:** `launchctl kickstart -k gui/$(id -u)/<label>`,
  then check the `.status` file updated. A job that works when you run it
  yourself can still be blocked when launchd runs it.

---

## Cloud layer: three routines on claude.ai

Set up at claude.ai/code/routines (or ask Claude Code to create them). Each
routine attaches the **OS** connector (plus your task tool and any business
data connectors for the brief).

**The pattern that keeps them maintainable:** the routine's own prompt is only
a pointer. The real instructions are a note in your vault, so one edit to the
note changes the routine and no pasted copy drifts. The weekly cleanup is the
exception: its method (Ben's optimizer, about 400 KB of references) lives in a
private git repo the routine clones.

**Cron times are UTC.** Convert from your local time, and re-check at every
daylight-saving change (set yourself two calendar reminders a year).

**Tools the routines may use on the connector:** `list_notes`, `search_notes`,
`read_note`, `edit_note`, `append_note`, `create_note`. Forbidden in every
routine prompt: `move_note`, `delete_note`, `delete_folder`, `manage_access`,
`set_access_default`.

**Before scheduling,** run each one once by hand and read what it wrote.

### Routine 1 - Daily Update (daily, 07:00 local, Sonnet is enough)

Trigger prompt (inside the routine):
```
You are <name>'s OS Daily Update agent. Your full instructions are a note in their OS.
1. With the OS connector, find `3 Resources/daily-update-prompt.md` (list_notes with pathPrefix
   "3 Resources/daily-update-prompt"), read it, and follow the instructions in it.
2. If you cannot reach the OS connector or that note, do not improvise: report
   "Daily Update could not reach the OS: <exact error>" and stop.
```

The note `3 Resources/daily-update-prompt.md`:
```
You are <name>'s Daily Update agent. One run, no questions, no memory. Everything you know comes
from their OS (the OS connector) and <task tool>. <name> reads the result on their phone.

## How to use the OS connector (read before any call)
- Find a note: list_notes with pathPrefix (exact path start, best) or search_notes (loose: verify
  relPath). Then read_note by docId.
- Edit: edit_note for a small change inside a note, append_note to add to the end, create_note for a
  new note. Re-read the note right before editing it.
- After every write, read_note it back and confirm your text is there. Retry once if not, then report.
- Never move_note, delete_note, delete_folder, manage_access or set_access_default.
- Say one short line before and after each batch of tool calls, so the session never sits silent.

## Steps
Use <timezone> time. "Yesterday" = the local calendar day before today.
1. Bootstrap (one batch): read AGENTS.md, Daily/<yesterday>.md, Daily/<yesterday>-capture.md (may be
   missing if the Mac was asleep: say so and continue), and Daily/<today>.md if it exists.
2. <Task tool>: which cards moved into Done yesterday. Count the Inbox and name overdue cards. Read only.
3. Fill yesterday's gaps: for each real decision, shipped thing or correction that has evidence but no
   line in Daily/<yesterday>.md, append one line:
   - (operator) 07:00 | <what happened> (unverified, source: <capture or card link>)
   At most 8 lines. Never repeat what is there in other words. Never log code progress. Nothing
   missing: write nothing.
4. Project pages: for each project the evidence touches, open its README.md and edit only the text
   under "## Auto (unverified)": up to 3 dated bullets, newest first, each with its source. A merged PR
   is "merged", never "live". Never touch any other part of a README, its reviewed: field, anything in
   Context/, or 1 Projects/_REGISTER.md. No such heading on the page: skip it and say so in the report.
5. Today's note: if Daily/<today>.md does not exist, create it (frontmatter type: daily, date; heading
   "# <today>"). Only if something needs <name> (a decision waiting, a client waiting, a failed job,
   task Inbox over 10), append one line per item:
   - (operator) 07:00 | Needs <name>: <item> (<link>)
6. Report in one phone-length summary: lines added, pages touched, Needs-you items, what you could not
   reach. Nothing changed: say "no new activity" and write nothing.

## Hard rules
- AGENTS.md binds you; where it and this prompt disagree, AGENTS.md wins.
- Never state an inference as fact. Everything you write is marked unverified with a source.
- Every link you write comes from a tool result in this run. Never guess a link.
- Never send anything to anyone. Never create a folder, never move or delete a note.
```

### Routine 2 - Weekly Cleanup (Sunday 06:00 local, Opus)

Setup:
1. Get Ben AI's **optimizer** skill from its author (see `skills.md`).
2. Make a private repo, e.g. `<you>/os-routines`, holding
   `.claude/skills/optimizer/` (the skill) and a `CLAUDE.md` with the house
   rules below.
3. Give Claude's cloud sessions access to these two private repos (connect
   GitHub for Claude Code on the web and allow both repos; check current docs
   for the exact screen). Note: the snapshot is refreshed by the Mac's hourly
   backup, so if the Mac slept all week the cleanup analyses an older copy.
   Its writes still go to the live vault through the connector.
4. The routine clones two repos as sources: `os-routines` and your backup repo
   (the hourly snapshot). It analyses the snapshot locally (fast, complete) and
   writes only through the connector. It never pushes.

**Before the first run,** add this exception to `AGENTS.md`, or the cleanup
will refuse to fix anything (the contract says audit changes need numbered
approval, and the contract wins): *"Exception: the weekly cleanup may apply up
to 10 mechanical fixes per run (a duplicate H1, an obvious missing
frontmatter field, a wiki link whose renamed target exists exactly once)."*

Trigger prompt:
```
You are the weekly cleanup for <name>'s OS. Nobody is watching; do not ask questions.
1. Read /home/user/os-routines/CLAUDE.md first. Its house rules bind you.
2. Then read /home/user/os-routines/.claude/skills/optimizer/SKILL.md and follow it as adapted by those
   house rules, reading its reference files as it directs.
3. The vault to analyse is /home/user/os-backup/OS (read-only snapshot). Every write goes through the
   OS connector, and only what the house rules allow: at most 10 mechanical fixes plus one report note.
4. If the connector is unreachable, write nothing, and report the exact error.
```

House rules (`os-routines/CLAUDE.md`):
```
# OS routines - weekly cleanup (unattended)
Where these rules and the skill disagree, these win; where they and the vault's AGENTS.md disagree,
AGENTS.md wins.

## Where things are
- os-backup/OS/: read-only snapshot, up to ~1 hour old. Analyse here. Its .context/ is absent on
  purpose: treat the vault as SYNCED anyway.
- OS connector: the live vault. Every write goes through it. list_notes + pathPrefix, read_note right
  before changing, edit_note, read back to verify.

## How the optimizer runs here
1. Discovery, roles, all frameworks, score, architectural read, on the snapshot. Skip task lists and
   checkpoint questions (nobody is there).
2. No user: save everything to plan, except up to 10 mechanical fixes applied live through the
   connector: a duplicate H1, a punctuation rule AGENTS.md sets, a missing frontmatter field whose value is obvious from the
   note itself, a wiki link whose target was renamed and exists exactly once. Never more than 10.
   Never anything in Context/, never a reviewed: field, never 1 Projects/_REGISTER.md.
3. Never move, rename or delete a note, never create or delete a folder, never change access. Moves go
   into the plan as "move_note <from> -> <to>" for the owner to approve; a later session runs them.
4. The plan is one new note: 2 Areas/personal/log/<YYYY-MM-DD>-weekly-cleanup.md. Contents,
   phone-readable:
   - Health score this week, last week's (from the previous weekly-cleanup note) and the change.
   - Applied (max 10): each with path and one line.
   - Approve list: numbered, smallest blast radius first, at most 15 items, each one line plus the
     exact action. The owner answers "approve all" or "all but 3 and 5" from any AI.
   - Also checked: empty notes, business facts that crept into skills, stale current pages (reviewed:
     older than 30 days), MAP drift (folders with no MAP row, MAP rows pointing at missing paths).
   - Architectural read: 5 lines max.
5. No HTML report (the OS is markdown only).
6. End with a push-length summary: score, applied count, approve-list count.

## Never
Write to the snapshot or push to any repository. Add a top-level folder. Send anything to anyone.
```

When you answer "approve all" (or "all but 3 and 5") in any Claude chat, that
Claude applies the list through the connector and logs one Daily line.

### Routine 3 - Weekly Brief (Sunday 08:00 local, Opus)

Trigger prompt: the same pointer pattern, to `3 Resources/operator-prompt.md`,
with one difference: if the OS is unreachable, still deliver what it can from
the other connectors and say so in the first line.

The note `3 Resources/operator-prompt.md`:
```
You are <name>'s Weekly Operator. You run once a week, in a fresh session with no memory. Everything
you know comes from the vault. <One line about the owner and businesses>. They read this on their phone.

## 1. Read first, in this order (OS connector: list_notes + pathPrefix, then read_note)
1. AGENTS.md (its rules bind you)  2. Context/business.md, Context/strategy.md
3. 1 Projects/_REGISTER.md  4. Daily/ - every note from the last 8 days
5. 2 Areas/personal/os-health.md  6. 0 Inbox/
7. 2 Areas/personal/log/<today>-weekly-cleanup.md
Do not read the whole vault.

## 2. Gather
State the date window you used.
- <Business data source> - <the numbers that matter>. Exclude test records and say how many.
- <Calendar> - last 7 days and the week ahead.
- <Task tool> - Inbox count and the age of the oldest card.
If a source is unreachable, say so in one line and carry on. A partial brief beats none.

## 3. Write the brief
A NEW note Daily/<today>-brief.md (create_note). Read it back. Under 500 words, phone-readable:
- Moved - projects with real activity. Name what shipped.
- Stalled - contradictions between the register and what Daily notes and tasks show. Name the wrong
  rows; never set a status yourself. Anything silent for 3 months or more.
- <Business section> - <the one number that matters>.
- Goal - progress toward <12-month target>; honest even when bad.
- System - from os-health.md; if older than 3 days, say so.
- Cleanup - health score and change, what it fixed, and its numbered approve list copied as is.
- Inbox - vault 0 Inbox/ count and task Inbox count, never merged.
- Priorities - propose 3 priorities for the week; <name> confirms in one line.
- One thing - the single highest-value action for the week.

## 4. Deliver
Report the brief in the session so it reaches the phone. The vault note is the permanent record.

## Hard rules
Never file a task outside Inbox. Never state an inference as fact. Never send anything to anyone.
Never edit Context/ or playbooks; propose instead. Never touch code. If a section has nothing real,
write "nothing this week". Direct tone, no preamble.
```

---

## Gotchas collected
1. Background jobs silently blocked from protected folders: the readability
   probe plus OSJobs.
2. A backup that logs OK while failing: check exit codes, count notes, alarm on
   staleness.
3. A half-loaded vault mirrored over a good backup: the loss guard.
4. Green dashboards for jobs that never ran: heartbeat files graded by age.
5. A script editing a human's note: each job owns its own file.
6. UTC cron drift at daylight-saving changes.
7. The contract blocking the cleanup's fixes: write the exception into
   `AGENTS.md` up front.
8. Wiki links inside tables (`[[path\|label]]`) confuse naive link checkers.
9. Chat-only work leaves no trace on the Mac: the "log this" habit is the only fix.
