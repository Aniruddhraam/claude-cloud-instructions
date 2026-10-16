# claude-cloud-instructions

Working instructions for Claude Code cloud sessions: claude.ai/code, the Code tab in the Claude app, `claude --cloud` and routines.

A cloud session runs on a fresh VM and never loads my local `~/.claude/CLAUDE.md` or `~/.claude/settings.json`. [`claude/CLAUDE.md`](claude/CLAUDE.md) carries the same rules, adapted to the cloud VM. It covers commit identity and no attribution, pushing through the session's GitHub proxy instead of my local signing tool, the tools the VM has, presentation defaults, and which skills exist there.

## Use it in a cloud session

Start the session's prompt with:

> Before anything else, fetch https://raw.githubusercontent.com/Aniruddhraam/claude-cloud-instructions/main/claude/CLAUDE.md with curl and follow it as your working instructions for this session.

`raw.githubusercontent.com` is on the cloud environments' default Trusted network list, so this works from a session on any repository.

## Load it automatically (optional)

Instead of asking each time, add this setup script to a cloud environment (claude.ai/code, environment settings). Claude Code then loads the file as user instructions in every session that uses the environment:

```bash
#!/bin/bash
mkdir -p ~/.claude
curl -fsSL https://raw.githubusercontent.com/Aniruddhraam/claude-cloud-instructions/main/claude/CLAUDE.md -o ~/.claude/CLAUDE.md
```

The environment caches what the setup script writes, so an update to the file reaches new sessions when the cache rebuilds: about every seven days, or right away if you edit the setup script. Run `/context` in a session to confirm `/root/.claude/CLAUDE.md` is listed under Memory files.
