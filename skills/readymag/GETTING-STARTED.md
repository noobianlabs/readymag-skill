# Use the Readymag skill

This folder teaches an AI agent how to inspect, troubleshoot, and edit Readymag
sites. You describe the task; the agent reads the instructions.

## Load it

Keep `SKILL.md` and `references/` together. Tell your agent:

> Read `/path/to/readymag/SKILL.md` and use the relevant reference files.
> Help me with this Readymag task: [describe what you want].

Replace the path with the actual location of this folder. Ask the agent to
confirm which file it loaded. If it accepts attachments instead of local files,
attach `SKILL.md` and the references needed for your task.

For automatic discovery and agent-specific installation paths, see the
[repository instructions](https://github.com/noobianlabs/readymag-skill#2-install-from-github).
No Claude plugin or special slash command is required for manual loading.

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
