# Readymag skill: getting started

A skill teaches Claude Code how to do one kind of job well. This one is for
**Readymag sites**: checking how a site looks on desktop, tablet and phone,
finding why something breaks, and making careful edits in the Readymag editor.
Everything in it was learned on a real site (Yasmeen Afifi's portfolio,
Aug–Oct 2026), and every rule says how it was checked.

You don't need to read the skill. Claude reads it. You describe the task in
plain words.

---

## 1. What you need

| To do this | You need |
|---|---|
| Ask questions, audit a **published** site | Claude Code (desktop app or terminal) + this skill. No Readymag login |
| Inspect or **edit** a project | The above, plus Chrome signed in to my.readymag.com and the Claude in Chrome extension connected |
| Check on a real iPad / iPhone size | A Mac with Xcode's iOS Simulator (optional; Claude can drive it) |

## 2. Install (2 minutes)

Full guide: **https://github.com/noobianlabs/readymag-skill**

**Easiest:** in the Claude desktop app's **Code** tab, paste:

> Install the Readymag plugin from github.com/noobianlabs/readymag-skill:
> run `claude plugin marketplace add noobianlabs/readymag-skill` and then
> `claude plugin install readymag@readymag-skill`.

**Or type it yourself** in a Claude Code session:

```
/plugin marketplace add noobianlabs/readymag-skill
/plugin install readymag@readymag-skill
```

**Or copy the folder:** put this `readymag` folder (the one with `SKILL.md`)
at `~/.claude/skills/readymag/` for every project, or at
`<your project>/.claude/skills/readymag/` for one project only.

**Shortcut command** `/readymag-akh <task>` loads the skill and runs the task.
It comes with the plugin (it may show as `/readymag:readymag-akh`).

**Check it worked:** start a new Claude Code session and type `/readymag`.
If it shows up in the list, it is installed. You can also just ask a Readymag
question; Claude picks the skill up by itself.

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
  (`my.readymag.com/edit/<number>/…`). Claude will not touch any other.
- **Name the page and the layout**: desktop, tablet or phone. Each page has
  its own three layouts.
- **Say what "done" looks like**: "like the desktop", "like the live site",
  "nothing overlaps on an iPhone SE".
- **One change at a time** for anything visual. Look at it, then ask for the
  next one.

## 5. Safety rules the skill follows (and you should too)

- **Duplicate first.** Edit a copy of the project, never the published one.
  Readymag has no reliable undo.
- **Backup before every write.** Claude saves the old values to a file and
  writes down how to undo each change.
- **Publishing is yours.** Claude can walk you through it but does not press
  Publish or connect domains unless you ask, step by step.
- **Keep a log.** In the client folder, `CONTEXT.md` (the rules for this
  project) and `CHANGELOG.md` (what changed, and how to undo it). Claude reads
  both before starting.

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
| `references/stacked-poster-scroll.md` | How to build the stacked-posters scroll effect |
| `GETTING-STARTED.md` | This guide |

## 8. When something goes wrong

| You see | Do this |
|---|---|
| Claude says the Readymag login page is showing | Sign in to my.readymag.com in Chrome yourself, then say "go on" |
| "The Chrome extension isn't connected" | Open Chrome, click the Claude extension, make sure it is on and connected |
| It looks right in the editor but wrong on your phone | Normal for touch and Safari issues. Ask Claude to check in the iOS Simulator, or publish with a password and look on the real device |
| "The site is down" | Often your network, not the site. Ask Claude to check from outside |
| A change went wrong | Ask "undo the last change": the backup and undo steps are in `CHANGELOG.md` |

## 9. Keeping it current

At the end of each Readymag session, ask Claude: **"add what we learned to the
Readymag skill."** New facts go into `SKILL.md` (short rules) or a file in
`references/` (details). If you keep a second copy of the skill in a project
folder, sync it:

```bash
rsync -a --delete ~/.claude/skills/readymag/ "<project>/.claude/skills/readymag/"
```
