# Readymag skill for Claude

A skill teaches Claude how to do one kind of job well. This one is for
**[Readymag](https://readymag.com) sites**. It can check how a site looks on
desktop, tablet and phone, find out why something breaks, and make careful
edits in the Readymag editor.

All of it was learned on a real portfolio site (Aug–Oct 2026), and each rule
records how it was checked. You don't have to read the skill, because Claude
does. You describe the task in plain words.

**What it helps with**

- Text that shows on desktop but is **invisible on iPad or iPhone**
- **Scroll animations or carousels** that never finish
- Making a **tablet or phone layout** "look like desktop, just smaller"
- **Fixed menus and headers** that move or disappear
- **Page heights**, layers, custom code pop-ups, Safari text-size bugs
- **Publishing** and connecting a **custom domain**, step by step

---

## 1. What you need

| To do this | You need |
|---|---|
| Ask questions, or audit a **published** site | [Claude Code](https://claude.com/claude-code): the **Code** tab of the Claude desktop app, or the `claude` terminal app. No Readymag login needed |
| Inspect or **edit** a project | All of the above, plus **Google Chrome** signed in to [my.readymag.com](https://my.readymag.com) with the **[Claude in Chrome](https://claude.com/chrome)** extension connected |
| Test on an iPad or iPhone screen size | Optional: a Mac with Xcode's iOS Simulator. Claude can drive it |

A paid Claude plan (Pro or Max) includes Claude Code.

---

## 2. Install the skill (pick one)

### Option A: ask Claude to do it (easiest)

Open the Claude desktop app, go to the **Code** tab, start a session in any
folder and paste this:

> Install the Readymag plugin from github.com/noobianlabs/readymag-skill for
> my user account: run `claude plugin marketplace add noobianlabs/readymag-skill`
> and then `claude plugin install readymag@readymag-skill`. Tell me when it's done.

Approve the commands when Claude asks. Then **start a new session** so the
skill loads.

### Option B: type two commands yourself

In a Claude Code session, type these one at a time:

```
/plugin marketplace add noobianlabs/readymag-skill
```

```
/plugin install readymag@readymag-skill
```

You can also run them in the Terminal app:

```bash
claude plugin marketplace add noobianlabs/readymag-skill
```

```bash
claude plugin install readymag@readymag-skill
```

Start a new session afterwards.

### Option C: copy the folder (no plugin system)

Download this repo (green **Code** button → **Download ZIP**) and unzip it.
Then copy the `skills/readymag` folder to `~/.claude/skills/readymag`. Or run:

```bash
git clone https://github.com/noobianlabs/readymag-skill.git /tmp/readymag-skill && mkdir -p ~/.claude/skills && cp -R /tmp/readymag-skill/skills/readymag ~/.claude/skills/
```

### Option D: Claude.ai chat or the desktop app's Chat tab

Zip the `skills/readymag` folder, so the zip holds a `readymag` folder with
`SKILL.md` inside. In Claude's **Settings**, find **Skills** (under Capabilities) and upload it. Chat can
explain things and help you plan. Editing a project needs Claude Code plus
Chrome (Option A or B).

### Check it worked

Start a **new** Claude Code session and type `/readymag`. The skill should
appear in the list (as `readymag` or `readymag:readymag`). Or ask any
Readymag question. Claude loads the skill on its own when the topic comes up.

---

## 3. Get started: your first three prompts

Copy one and change the parts in [brackets].

**1. Audit a site (no login)**

> Audit https://[your-site.com]. List every page, say which ones have tablet
> and phone layouts, and flag anything that will look broken on a phone.

**2. Explain a problem**

> On the iPad the text on my [Selected Work] page is invisible. Why?

(The usual cause is a hover animation that starts at 0% opacity. Touch
screens have no hover, so the text never appears.)

**3. Make one change on a copy**

> In project [1234567] (my working copy), set the tablet height of page [3]
> to [2700] and check the scroll animation still finishes.

The project number is in the editor's address bar:
`my.readymag.com/edit/`**`1234567`**`/…`

**Shortcut command:** `/readymag-akh [task]` loads the skill and starts the
task in one go. In the plugin it may show as `/readymag:readymag-akh`.

---

## 4. How to ask so it goes well

- **Name the working copy.** Give its project number. Claude won't touch any
  other project.
- **Name the page and the layout** (desktop, tablet or phone). Each page has
  its own three layouts.
- **Say what "done" looks like**, e.g. "like the desktop", "like the live
  site", "nothing overlaps on an iPhone SE".
- **One visual change at a time.** Look at it, then ask for the next one.
- **Say "add what we learned to the Readymag skill"** at the end of a
  session. That keeps it getting smarter.

## 5. Safety rules (the skill follows them, and so should you)

- **Duplicate first.** Edit a copy of your project, never the published one.
  Readymag has no reliable undo.
- **Back up before every change.** Claude saves the old values to a file and
  writes down how to undo each change.
- **You press Publish.** Claude can walk you through publishing and domains,
  but it doesn't publish unless you tell it to, step by step.
- **Keep a log.** In your project folder, keep `CONTEXT.md` (the rules for
  this site) and `CHANGELOG.md` (what changed and how to undo it). Claude
  reads both before it starts.

## 6. Five things about Readymag that surprise everyone

1. **It's adaptive, not responsive.** It picks the desktop, tablet or phone
   layout once, when the page loads, and after that it only scales.
   Squeezing the browser window is not a phone test.
2. **The editor preview shows the layout the editor is on**, not your window
   size. Switch the editor to Tablet or Mobile first.
3. **Scroll animations need room.** If the page is too short, a carousel
   never finishes. Check the page height first.
4. **Hover doesn't exist on touch screens.** Anything that appears on hover
   stays invisible on iPad and iPhone unless it has its own touch setting.
5. **iPad Safari enlarges small text by itself.** Custom code that copies
   text sizes needs a fix, and the skill has it.

## 7. When something goes wrong

| You see | Do this |
|---|---|
| Claude says the Readymag login page is showing | Sign in to my.readymag.com in Chrome yourself, then say "go on" |
| "The Chrome extension isn't connected" | Open Chrome, open the Claude side panel, check you're signed in and click **Connect** |
| `/readymag` doesn't appear | Start a new session. If it still doesn't show, run the install again |
| It looks right in the editor but wrong on your phone | Common with touch and Safari issues. Ask Claude to check in the iOS Simulator, or publish with a password and look on the real device |
| "The site is down" | Often it's your network, not the site. Ask Claude to check from outside |
| A change went wrong | Say "undo the last change". The backup and the undo steps are in `CHANGELOG.md` |

## 8. Updating

Plugin install, in the Terminal app:

```bash
claude plugin marketplace update readymag-skill
```

```bash
claude plugin update readymag@readymag-skill
```

Then start a new session. (Or ask Claude in the Code tab to run those two.)

Folder install: download it again and replace `~/.claude/skills/readymag`.

## What's in this repo

| Path | What it is |
|---|---|
| `skills/readymag/SKILL.md` | The core: engine facts, editor recipes, how to verify, common mistakes |
| `skills/readymag/references/` | Details: the project data model, custom code and iPad/iPhone fixes, rescaling desktop to tablet, the stacked-covers scroll effect |
| `skills/readymag/GETTING-STARTED.md` | A copy of this guide that travels with the skill |
| `commands/readymag-akh.md` | The `/readymag-akh` shortcut |
| `.claude-plugin/` | Plugin and marketplace files used by `/plugin install` |

## License

MIT. See [LICENSE](LICENSE).
