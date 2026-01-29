# 🦞 Lobster Skill for Clawdbot

A [Clawdbot](https://github.com/clawdbot/clawdbot) skill for **Lobster** — the workflow runtime that brings deterministic pipelines with approval gates to AI agents.

## What is Lobster?

Lobster is a workflow engine that lets Clawdbot run multi-step automations that:
- **Halt and ask** before taking irreversible actions
- **Remember state** between runs (no re-processing yesterday's data)
- **Save tokens** by running deterministic pipelines instead of re-planning each step

Think of it as **Zapier for AI agents, but with human checkpoints**.

## Install

```bash
# Install the skill
clawdhub install lobster

# Clone the Lobster runtime
git clone https://github.com/moltbot/lobster ~/tools/lobster
cd ~/tools/lobster && npm install && npx tsc
```

## Quick Start

```bash
LOBSTER="node ~/tools/lobster/bin/lobster.js"

# Simple pipeline
$LOBSTER 'exec --json --shell "echo [1,2,3]" | head --n 2 | json'

# With approval gate
$LOBSTER run --mode tool 'exec --json --shell "echo [1,2,3]" | approve --prompt "Process these?"'

# Monitor a GitHub PR for changes
$LOBSTER "workflows.run --name github.pr.monitor --args-json '{\"repo\":\"owner/repo\",\"pr\":123}'"
```

## Features

| Feature | Description |
|---------|-------------|
| **Typed Pipelines** | JSON-first data flow, not text pipes |
| **Approval Gates** | `approve` halts execution until human confirms |
| **Stateful** | `diff.last` and `state.*` track what's been processed |
| **GitHub Integration** | Built-in PR monitoring workflows |
| **Clawdbot Native** | Call Clawdbot tools via `clawd.invoke` |

## Core Commands

| Command | Purpose |
|---------|---------|
| `exec --json --shell "cmd"` | Run shell command, parse JSON output |
| `where 'field=value'` | Filter objects |
| `pick field1,field2` | Select fields |
| `head --n 5` | Take first N items |
| `approve --prompt "ok?"` | **Halt for human approval** |
| `diff.last --key "mykey"` | Compare to last run |
| `json` / `table` | Render output |

## Contributing

PRs welcome! 

1. Fork this repo
2. Make your changes to `SKILL.md`
3. Submit a PR

After merge, the maintainer will publish to ClawdHub:
```bash
clawdhub publish . --slug lobster --version X.Y.Z
```

## Links

- **Lobster Runtime:** [github.com/moltbot/lobster](https://github.com/moltbot/lobster)
- **ClawdHub:** [clawdhub.com](https://clawdhub.com)
- **Clawdbot:** [github.com/clawdbot/clawdbot](https://github.com/clawdbot/clawdbot)

## License

MIT
