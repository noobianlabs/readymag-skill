# Use the Readymag skill

This folder teaches an AI agent how to inspect, troubleshoot, and edit Readymag
sites. You describe the task; the agent reads the instructions.

## Copy and paste into your agent

Paste this whole message into your AI agent:

```text
Load the Readymag skill from:
https://github.com/noobianlabs/readymag-skill

Read skills/readymag/SKILL.md and load its reference files as needed.
If you can use git, clone the repository into an unused local folder;
otherwise read the files from GitHub. Confirm when the skill is loaded,
then ask what I want to do with my Readymag site.
If you cannot access the files, tell me what to download and attach.
```

Then describe your task—for example: **“Check my site on tablet and phone:
[your site URL].”** No file paths or special commands to figure out.

If your agent cannot access GitHub, click **Code → Download ZIP**, unzip it,
and give the agent the `skills/readymag` folder or attach `SKILL.md` and the
references it needs. Loading applies to the current conversation; for
installation that persists across sessions, use the per-agent instructions
[below](https://github.com/noobianlabs/readymag-skill#2-install-from-github).

## Try a task

- **Explain:** “Why does my Readymag layout look different when I resize the window?”
- **Audit:** “Check https://[your-site.com] on desktop, tablet, and phone. Report
  problems and distinguish verified behavior from checks you could not perform.”
- **Edit:** “In working-copy project [1234567], page 3, tablet layout, set the page
  height to 2700 and verify that the scroll animation finishes. Back up the old
  values first.”

For edits, give the project number from `my.readymag.com/edit/<number>/…`, the
page, the layout, and the desired result. Use a duplicate project by default.
Publishing and domain changes require your authorization.

## Tools you need

- **Questions:** an agent that can read the skill.
- **Public-site inspection:** web access. Screenshots and real browser input
  are needed to verify appearance and interactions.
- **Editor changes:** browser control and your signed-in Readymag session.
  Sign in yourself. The agent should save old values before changing them.
- **iOS-specific checks:** a real device or iOS Simulator, when available.

The skill does not install browser tools. If a tool is missing, the agent can
explain what it can check and provide manual steps for the remaining work.

## Update it

Pull or download the latest repository, preserve any local notes, and replace
your installed skill folder. Keep backups outside folders scanned for skills.
Start a new agent session to load the updated version.
