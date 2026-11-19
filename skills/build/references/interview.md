# The interview

The `Context/` files this interview produces are the most valuable part of the
whole OS. Every AI reads them before it writes anything for the person, so a
good hour here saves hundreds of corrections later.

## How to run it

- **Seven short sections, one file each.** Ask a section at a time, three to
  five questions in one message. The person answers **by voice** (dictation on
  the phone or the Mac keyboard's microphone), in any order, as long or short as
  they like. Skipping is fine. Rambling is good: you do the structuring.
- **Read before you ask.** If the person has existing notes, a website, a pitch
  deck or old docs, read them first and ask only what they do not answer. Say
  what you already know in one line so they can correct it.
- **After each section, write the file immediately** (sessions end without
  warning), then show a 5-line summary and ask "what did I get wrong?"
- **Keep their words.** Put striking phrases verbatim under a heading
  `## In their own words (interview, <date>)`. Never polish their voice into
  marketing copy.
- **Mark doubt.** Anything said hesitantly or inferred by you gets `(?)` and a
  line in a `## Still to confirm` list at the bottom of the file. Never fill a
  gap with a plausible guess.
- **No placeholders.** A section with nothing in it is left out, or says
  "EMPTY ON PURPOSE" and what would fill it.
- **Close the loop.** At the end, go through every `(?)` with the person. Then
  set `status: active` and `reviewed: <today>` on each Context file. A file is
  finished only when it has no `(?)` left. (In the build this kit comes from,
  skipping this step left the files in draft for weeks.)

The questions below are prompts. Pick the ones that matter for this person and
let the conversation go where it is useful.

## 1. Who you are -> `Context/me.md`
- In two sentences, what do you do, and for whom?
- What is your background, and what are you unusually good at?
- How do you like to work: hours, places, solo or with people, fast or careful?
- How should an AI talk to you? (Level of detail, tone, what annoys you,
  whether to push back.)
- What do AIs usually get wrong about you or your work?
- Anything personal you want every AI to keep in mind (family, health,
  location, time zone, languages)?

## 2. What you run -> `Context/business.md`
- List every business, job or revenue line. For each: what it is, for whom,
  how far along it is, and roughly what it earns or costs.
- Who is involved: partners, staff, freelancers, key contacts. Who decides what?
- How does the money actually move (pricing model, payment tools, invoicing)?
- Which numbers tell you whether a business is healthy?

## 3. Your customers -> `Context/icp.md`
- Describe your best customer as a real person (first name only). Why are they
  the best?
- What problem brings people to you, in the words *they* use?
- What happens right before someone buys? Where do leads really come from?
- What objections do you hear, and what answer works?
- Who is the customer you do not want, and how do you spot them early?

## 4. How you sound -> `Context/brand.md`
- How would you describe your tone to a new copywriter in three words?
- Paste or point to something you wrote that sounds exactly like you.
- Words, phrases or claims you never want used. Topics you avoid.
- What is your brand deliberately *not*?
- If more than one person writes (you, a partner), how do the voices differ?
- Where do logos, colours, fonts and photos live?

## 5. Where you are going -> `Context/strategy.md`
- What has to be true in 12 months for this year to count as a win? Numbers
  if you have them.
- And in three years?
- What are you deliberately saying no to right now?
- What constraints are fixed (money, time, location, commitments)?
- Which open questions, once answered, would change your plans?

## 6. Your tools -> `Context/infrastructure.md`
- Which tools do you use for email, chat, calendar, tasks, files, CRM,
  payments, website, code, automation?
- For each kind of information, which system is the one source of truth?
  (Customers, payments, tasks, files, contracts.)
- What did you stop using, so no AI suggests it again?
- Do you have a server, a domain, a GitHub account? (Feeds the Baalda phase.)

## 7. What is in motion -> `1 Projects/`
- List what you are working on now. For each: what it is, for whom, what
  "done" looks like, a deadline if there is a real one, and where its files and
  code live.
- Anything you started and dropped, so nobody restarts it by accident?
- What repetitive work eats your week? ("When X happens I do Y, it takes Z.")

Each active project gets a folder with `README.md`, `decisions.md` and
`open-questions.md` from `vault-templates.md`, and a row in
`1 Projects/_REGISTER.md`. Dropped projects get a register row with status
`paused` or `abandoned`. Repetitive work goes to `Notes/ideas/` as automation
candidates.

## Finish
- "Anything else I should read? A folder, a doc, a link."
- Write `Context/README.md`: one line per file.
- Close the loop (above), then one line in today's Daily note.

## For a company instead of a solo operator
Same sections, plus: the team and who owns what, departments, goals per team,
and which folders each person should be able to see. Baalda supports
per-folder access; add people only after two weeks of the owner using it alone.
