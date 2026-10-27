# Readymag skill

A skill teaches an AI coding agent how to do one kind of job well. This one is
for **[Readymag](https://readymag.com) sites**: checking how a site looks on
desktop, tablet and phone, finding why something breaks, and making careful
edits in the Readymag editor.

All of it was learned on a real portfolio site (Aug–Oct 2026), and each rule
records how it was checked. You don't have to read the skill, because your
agent does. You describe the task in plain words.

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
| Ask questions, or audit a **published** site | An agent that can read this skill; auditing a URL also needs web access. No Readymag login needed |
| Inspect or **edit** a project | All of the above, plus a browser you control that is signed in to [my.readymag.com](https://my.readymag.com) and can run page JavaScript, take screenshots, and send real input |
| Test on an iPad or iPhone screen size | Optional: a Mac with Xcode's iOS Simulator. Requires simulator-control tools |

The skill supplies instructions, not a browser integration. Configure browser
automation separately for visual checks or edits; knowledge-only questions do
not require it. Use only operations supported by your integration.

---

## 2. Install from GitHub

Pick one of the two download methods, then copy `skills/readymag` into your
agent's skill directory (see the table below).

### Option 1: Download ZIP

1. Open https://github.com/noobianlabs/readymag-skill
2. Click **Code** → **Download ZIP**, then unzip it.
3. Copy the `skills/readymag` folder (the one containing `SKILL.md`) to your
   agent's skill directory.

### Option 2: Clone with git

```bash
git clone https://github.com/noobianlabs/readymag-skill.git
```

Then copy `readymag-skill/skills/readymag` to your agent's skill directory.

### Where to put it (by agent)

| Agent | Project scope | User scope (all projects) | Source |
|---|---|---|---|
| OpenCode | `.opencode/skills/readymag/` (also `.claude/skills/` or `.agents/skills/`) | `~/.config/opencode/skills/readymag/` (also `~/.claude/skills/` or `~/.agents/skills/`) | [opencode.ai/docs/skills](https://opencode.ai/docs/skills/) |
| Codex | `.agents/skills/readymag/` | `~/.agents/skills/readymag/` | [Codex skill locations](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills) |
| Claude Code | `.claude/skills/readymag/` | `~/.claude/skills/readymag/` | [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills) |

Notes:

- OpenCode searches the documented hidden directories above, up to the git
  worktree root; simply cloning this repository does not install the skill.
- Install one copy per agent; avoid multiple active copies with the same name.
- Claude Code also supports a plugin install (optional compatibility preserved
  in this repo):

  ```bash
  claude plugin marketplace add noobianlabs/readymag-skill
  claude plugin install readymag@readymag-skill
  ```

  Or type `/plugin marketplace add noobianlabs/readymag-skill` and
  `/plugin install readymag@readymag-skill` inside a Claude Code session.

### Copy it from the terminal

From inside the downloaded or cloned `readymag-skill` repository, choose one
user-level destination. These commands are for macOS/Linux shells:

```bash
# OpenCode (use this, or choose one alternative below)
readymag_dest="$HOME/.config/opencode/skills/readymag"
# Codex: readymag_dest="$HOME/.agents/skills/readymag"
# Claude Code: readymag_dest="$HOME/.claude/skills/readymag"

if [ -e "$readymag_dest" ] || [ -L "$readymag_dest" ]; then
  echo "Already installed: $readymag_dest. See Updating below."
else
  mkdir -p "$readymag_dest" && cp -R skills/readymag/. "$readymag_dest/"
fi
```

For a project install, set the destination to the project's absolute path
followed by the project-scope path in the table. Windows users can copy the
folder with File Explorer to the equivalent location in their user profile.
The final layout must be `readymag/SKILL.md` alongside `readymag/references/`,
not `readymag/readymag/SKILL.md`.

### Manual-loading fallback (other agents)

If your agent does not scan a skill directory, copy `skills/readymag` anywhere
convenient and load `SKILL.md` into the agent's context manually (paste it, or
point the agent at the file). Keep the `references/` folder next to `SKILL.md`
so relative references resolve. The frontmatter is standard Agent Skills
format, so most SKILL.md-aware tools accept it unchanged.

### Check it worked

Start a **new** session. In OpenCode, ask “Load the readymag skill and explain
how tablet layouts work.” In Codex, mention `$readymag`; in Claude Code, use
`/readymag` (the plugin may expose `/readymag:readymag`). Ask the agent to
confirm it loaded `SKILL.md` from the installed folder. Automatic selection
also depends on the agent's configuration; manual loading remains a fallback.

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

**Shortcut command (Claude Code plugin):** `/readymag-akh [task]` loads the
skill and starts the task in one go. In the plugin it may show as
`/readymag:readymag-akh`.

---

## 4. How to ask so it goes well

- **Name the working copy.** Give its project number. The agent won't touch any
  other project.
- **Name the page and the layout** (desktop, tablet or phone). Each page has
  its own three layouts.
- **Say what "done" looks like**, e.g. "like the desktop", "like the live
  site", "nothing overlaps on an iPhone SE".
- **One visual change at a time.** Look at it, then ask for the next one.
- **Say "add what we learned to the Readymag skill"** at the end of a
  session. That keeps it getting smarter.

## 5. Safety rules (the skill follows them, and so should you)

- **Duplicate first.** Default to a working copy; editing the original requires
  explicit direction. Do not rely on undo as the only recovery method.
- **Back up before every change.** The agent saves the old values to a file
  and writes down how to undo each change.
- **You press Publish.** The agent can walk you through publishing and
  domains, but it doesn't publish unless you authorize it.
- **Keep a log.** In your project folder, keep `CONTEXT.md` (the rules for
  this site) and `CHANGELOG.md` (what changed and how to undo it). The agent
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
| The agent says the Readymag login page is showing | Sign in to my.readymag.com in your browser yourself, then say "go on" |
| "The browser extension isn't connected" | Reconnect using your browser integration's setup instructions; side panels and Connect buttons are provider-specific |
| The skill is not discovered | Check the path and uppercase `SKILL.md`, restart the session, and check your agent's skill settings. Slash-command syntax varies by agent |
| It looks right in the editor but wrong on your phone | Common with touch and Safari issues. Ask the agent to check in the iOS Simulator, or publish with a password and look on the real device |
| "The site is down" | Often it's your network, not the site. Ask the agent to check from outside |
| A change went wrong | Say "undo the last change". The backup and the undo steps are in `CHANGELOG.md` |

## 8. Updating

For a cloned repository, run `git pull --ff-only` from the clone. For a ZIP,
download and extract a fresh copy. Before replacing your installed folder,
move the old copy outside every scanned skills directory and preserve any
local notes you want to keep. Then copy the fresh `skills/readymag` folder to
the original destination, using the install steps above, and start a new session.

Plugin install (Claude Code), in the terminal:

```bash
claude plugin marketplace update readymag-skill
claude plugin update readymag@readymag-skill
```

Then start a new Claude Code session.

## What's in this repo

| Path | What it is |
|---|---|
| `skills/readymag/SKILL.md` | The core: engine facts, editor recipes, how to verify, common mistakes |
| `skills/readymag/references/` | Details: the project data model, custom code and iPad/iPhone fixes, rescaling desktop to tablet, the stacked-covers scroll effect |
| `skills/readymag/GETTING-STARTED.md` | A short getting-started guide that travels with the skill |
| `commands/readymag-akh.md` | The `/readymag-akh` shortcut (Claude Code plugin) |
| `.claude-plugin/` | Plugin and marketplace files used by `/plugin install` (optional compatibility) |

## License

MIT. See [LICENSE](LICENSE).
