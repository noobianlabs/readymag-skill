# Readymag skill: getting started

A skill teaches an AI coding agent how to do one kind of job well. This one is
for **Readymag sites**: checking how a site looks on desktop, tablet and
phone, finding why something breaks, and making careful edits in the Readymag
editor. Everything in it was learned on a real site (Aug–Oct 2026), and every
rule says how it was checked.

You don't need to read the skill. The agent reads it. You describe the task in
plain words.

---

## 1. What you need

| To do this | You need |
|---|---|
| Ask questions, audit a **published** site | An agent that can read this skill; URL audits also need web access. No Readymag login |
| Inspect or **edit** a project | The above, plus a browser you control that is signed in to my.readymag.com and can run page JavaScript, take screenshots, and send real input |
| Check on a real iPad / iPhone size | A Mac with Xcode's iOS Simulator (optional; requires simulator-control tools) |

## 2. Install (2 minutes)

Full guide with per-agent paths: **https://github.com/noobianlabs/readymag-skill**

Download or clone the repo, then copy the `skills/readymag` folder (the one
with `SKILL.md`) into your agent's skill directory:

- **OpenCode**: `~/.config/opencode/skills/readymag/` (or `.opencode/skills/readymag/` in a project)
- **Codex**: `~/.agents/skills/readymag/` (or `.agents/skills/readymag/` in a project)
- **Claude Code**: `~/.claude/skills/readymag/` (or `.claude/skills/readymag/` in a project)

**Shortcut command** `/readymag-akh <task>` (Claude Code plugin only) loads the
skill and runs the task.

**Check it worked:** start a new session. Ask OpenCode to load `readymag`,
mention `$readymag` in Codex, or use `/readymag` in Claude Code. Ask it to
confirm the installed `SKILL.md` path. Other agents can read the file and
references directly. Browser control is configured separately; installing
the skill does not install an extension or grant browser access.

## 3. Your first three prompts

Copy one, change the bits in brackets.

1. **Audit, no login:**
   > Audit https://[your-site.com] — list every page, which ones have tablet
   > and phone layouts, and anything that will look broken on a phone.

2. **Explain a problem:**
   > On the iPad the text on my Selected Work page is invisible. Why?

   (Usual answer: a hover animation that starts at 0% opacity. Touch screens
   have no hover, so the text never appears.)

3. **Make a change on a copy:**
   > In project [1234567] (my working copy), set the tablet height of page 3
   > to 2700 and check the scroll animation still finishes.

## 4. How to ask so it goes well

- **Name the working copy.** Give the project number from the editor URL
  (`my.readymag.com/edit/<number>/…`). The agent will not touch any other.
- **Name the page and the layout**: desktop, tablet or phone. Each page has
  its own three layouts.
- **Say what "done" looks like**: "like the desktop", "like the live site",
  "nothing overlaps on an iPhone SE".
- **One change at a time** for anything visual. Look at it, then ask for the
  next one.

## 5. Safety rules the skill follows (and you should too)

- **Duplicate first.** Default to a working copy; editing the original requires
  explicit direction. Do not rely on undo as the only recovery method.
- **Backup before every write.** The agent saves the old values to a file and
  writes down how to undo each change.
- **Publishing is yours.** The agent can walk you through it but does not
  press Publish or connect domains unless you authorize it.
- **Keep a log.** In the client folder, `CONTEXT.md` (the rules for this
  project) and `CHANGELOG.md` (what changed, and how to undo it). The agent
  reads both before starting.

## 6. Five things that surprise everyone

1. **Readymag is adaptive, not responsive.** It picks desktop, tablet or phone
   once, when the page loads, then only scales. Squeezing the browser window
   is not a phone test.
2. **The editor preview shows the layout the editor is on,** not your window
   size. Switch the editor to Tablet or Mobile first.
3. **Scroll animations need room.** If the page is too short, a carousel never
   finishes. Check page height before anything else.
4. **Hover effects don't exist on touch.** Anything that appears on hover is
   invisible on iPad and iPhone unless it has its own touch setting.
5. **iPad Safari enlarges small text by itself.** Custom code that copies text
   sizes needs a fix (it's in the skill).

## 7. What's in the folder

| File | What's in it |
|---|---|
| `SKILL.md` | The core: engine facts, editor recipes, verification rules, common mistakes |
| `references/readymag-data-model.md` | Where every setting lives in the project data |
| `references/custom-code-ios-and-live-data.md` | Custom code, iPad/iPhone Safari fixes, the iOS Simulator, duplicating pages, reading the live site |
| `references/rescale-breakpoint.md` | Exact desktop to tablet/phone rescale recipe |
| `references/stacked-poster-scroll.md` | The stacked-covers scroll effect |
| `GETTING-STARTED.md` | This guide |

## 8. When something goes wrong

| You see | Do this |
|---|---|
| The agent says the Readymag login page is showing | Sign in to my.readymag.com in your browser yourself, then say "go on" |
| "The browser extension isn't connected" | Reconnect using your browser integration's setup instructions; not every integration uses an extension |
| It looks right in the editor but wrong on your phone | Normal for touch and Safari issues. Ask the agent to check in the iOS Simulator, or publish with a password and look on the real device |
| "The site is down" | Often your network, not the site. Ask the agent to check from outside |
| A change went wrong | Ask "undo the last change": the backup and undo steps are in `CHANGELOG.md` |

## 9. Keeping it current

At the end of each Readymag session, ask the agent: **"add what we learned to
the Readymag skill."** New facts go into `SKILL.md` (short rules) or a file in
`references/` (details). Keep one canonical copy and copy updates to the
installed location listed above. Preserve local notes before replacing any
installed copy; keep backups outside directories scanned for skills. See the
repository README for GitHub download, terminal copy, and update instructions.
