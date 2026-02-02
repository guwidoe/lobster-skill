---
name: lobster
description: >
  Lobster workflow runtime for deterministic pipelines with approval gates.
  Use when: (1) Running multi-step automations that need human approval before side effects,
  (2) Monitoring PRs/issues for changes, (3) Processing data through typed JSON pipelines,
  (4) Email triage or batch operations, (5) Any workflow that should halt and ask before acting.
  Lobster saves tokens by running deterministic pipelines instead of re-planning each step.
---

# Lobster

> **Contribute:** Source code & PRs welcome at [github.com/guwidoe/lobster-skill](https://github.com/guwidoe/lobster-skill)

Workflow runtime for AI agents — typed pipelines with approval gates.

## CLI Location

```bash
# Set alias (adjust path to your install location)
LOBSTER="node /home/molt/clawd/tools/lobster/bin/lobster.js"

# Or install globally: npm install -g @clawdbot/lobster
# Then use: lobster '<pipeline>'
```

## Quick Reference

```bash
# Run pipeline (human mode - pretty output)
$LOBSTER '<pipeline>'

# Run pipeline (tool mode - JSON envelope for integration)
$LOBSTER run --mode tool '<pipeline>'

# Run workflow file
$LOBSTER run path/to/workflow.lobster

# Resume after approval
$LOBSTER resume --token "<token>" --approve yes|no

# List commands/workflows
$LOBSTER commands.list
$LOBSTER workflows.list
```

## Core Commands

| Command | Purpose |
|---------|---------|
| `exec --json --shell "cmd"` | Run shell, parse stdout as JSON |
| `where 'field=value'` | Filter objects |
| `pick field1,field2` | Project fields |
| `head --n 5` | Take first N items |
| `sort --key field --desc` | Sort items |
| `groupBy --key field` | Group by key |
| `dedupe --key field` | Remove duplicates |
| `map --wrap key` | Transform items |
| `template --text "{{field}}"` | Render templates |
| `approve --prompt "ok?"` | **Halt for approval** |
| `diff.last --key "mykey"` | Compare to last run (stateful) |
| `state.get key` / `state.set key` | Read/write persistent state |
| `json` / `table` | Render output |

## Built-in Workflows

```bash
# Monitor PR for changes (stateful - remembers last state)
$LOBSTER "workflows.run --name github.pr.monitor --args-json '{\"repo\":\"owner/repo\",\"pr\":123}'"

# Monitor PR and emit message only on change
$LOBSTER "workflows.run --name github.pr.monitor.notify --args-json '{\"repo\":\"owner/repo\",\"pr\":123}'"
```

## Approval Flow (Tool Mode)

When a pipeline hits `approve`, it returns:

```json
{
  "status": "needs_approval",
  "requiresApproval": {
    "prompt": "Send 3 emails?",
    "items": [...],
    "resumeToken": "eyJ..."
  }
}
```

To continue:
```bash
$LOBSTER resume --token "eyJ..." --approve yes
```

## Example Pipelines

```bash
# List recent PRs, filter merged, show as table
$LOBSTER 'exec --json --shell "gh pr list --repo owner/repo --json number,title,state --limit 20" | where "state=MERGED" | table'

# Get data, require approval, then process
$LOBSTER run --mode tool 'exec --json --shell "echo [{\"id\":1},{\"id\":2}]" | approve --prompt "Process these?" | pick id | json'

# Diff against last run (only emit on change)
$LOBSTER 'exec --json --shell "gh pr view 123 --repo o/r --json state,title" | diff.last --key "pr:o/r#123" | json'
```

## Workflow Files (.lobster)

YAML/JSON files with steps, conditions, and approval gates:

```yaml
name: pr-review-reminder
steps:
  - id: fetch
    command: gh pr list --repo ${repo} --json number,title,reviewDecision
  - id: filter
    command: jq '[.[] | select(.reviewDecision == "")]'
    stdin: $fetch.stdout
  - id: notify
    command: echo "PRs needing review:" && cat
    stdin: $filter.stdout
    approval: required
```

Run: `$LOBSTER run workflow.lobster --args-json '{"repo":"owner/repo"}'`

## Clawdbot Integration

Lobster can call Clawdbot tools via `clawd.invoke`:

```bash
$LOBSTER 'clawd.invoke --tool message --action send --args-json "{\"target\":\"123\",\"message\":\"hello\"}"'
```

Requires `CLAWD_URL` and `CLAWD_TOKEN` environment variables.

## Complete Example: Email Triage Workflow

This example shows a practical email triage workflow that:
1. Fetches unread emails via Clawdbot's email tools
2. Filters by sender domain
3. Requires approval before taking action
4. Marks emails as processed using stateful tracking

### The Pipeline

```bash
# Email triage: fetch unread → filter important → approve → mark read
$LOBSTER run --mode tool '
  clawd.invoke --tool email --action list --args-json "{\"folder\":\"inbox\",\"unread\":true,\"limit\":20}" |
  where "from contains @important-client.com" |
  diff.last --key "email:triage:important-client" |
  approve --prompt "Mark these emails as triaged?" |
  clawd.invoke --tool email --action mark_read --stdin-ids
'
```

### Step-by-Step Breakdown

**Step 1: Fetch unread emails**
```bash
clawd.invoke --tool email --action list --args-json "{\"folder\":\"inbox\",\"unread\":true,\"limit\":20}"
```
Returns JSON array: `[{"id":"abc","from":"alice@important-client.com","subject":"Q3 Review"}, ...]`

**Step 2: Filter by sender**
```bash
where "from contains @important-client.com"
```
Keeps only emails from the important client domain.

**Step 3: Diff against last run**
```bash
diff.last --key "email:triage:important-client"
```
Compares current results to last run. Only emits items that are **new** since last triage. This prevents re-processing the same emails.

**Step 4: Approve before action**
```bash
approve --prompt "Mark these emails as triaged?"
```
Pipeline halts here. Returns:
```json
{
  "status": "needs_approval",
  "requiresApproval": {
    "prompt": "Mark these emails as triaged?",
    "items": [{"id":"abc","from":"alice@important-client.com","subject":"Q3 Review"}],
    "resumeToken": "eyJ..."
  }
}
```

The agent presents this to the human. Human says yes/no.

**Step 5: Resume and execute**
```bash
$LOBSTER resume --token "eyJ..." --approve yes
```
If approved, continues to the `mark_read` action.

### Workflow File Version

For reusable workflows, save as `email-triage.lobster`:

```yaml
name: email-triage
description: Triage emails from important senders
args:
  sender_domain:
    required: true
    description: Domain to filter (e.g., "@client.com")
  folder:
    default: inbox

steps:
  - id: fetch
    run: |
      clawd.invoke --tool email --action list \
        --args-json '{"folder":"${folder}","unread":true,"limit":50}'
    
  - id: filter
    stdin: $fetch
    run: where "from contains ${sender_domain}"
    
  - id: diff
    stdin: $filter
    run: diff.last --key "email:triage:${sender_domain}"
    
  - id: approve
    stdin: $diff
    run: approve --prompt "Process ${count} emails from ${sender_domain}?"
    skip_if_empty: true
    
  - id: mark
    stdin: $approve
    run: |
      clawd.invoke --tool email --action mark_read --stdin-ids
```

Run it:
```bash
$LOBSTER run email-triage.lobster --args-json '{"sender_domain":"@important-client.com"}'
```

### Key Patterns Demonstrated

| Pattern | Commands Used | Purpose |
|---------|---------------|---------|
| **Stateful diffing** | `diff.last --key` | Only process new items since last run |
| **Approval gate** | `approve --prompt` | Human confirms before side effects |
| **Tool integration** | `clawd.invoke` | Call Clawdbot email tools from pipeline |
| **Filtering** | `where` | Narrow down to relevant items |

### Adapting for Other Use Cases

**Newsletter cleanup:**
```bash
$LOBSTER '... | where "from contains @newsletter" | approve --prompt "Archive these?" | clawd.invoke --tool email --action archive --stdin-ids'
```

**Flag for follow-up:**
```bash
$LOBSTER '... | where "subject contains URGENT" | approve --prompt "Flag these as important?" | clawd.invoke --tool email --action flag --stdin-ids'
```

---

## Troubleshooting

### "Command not found" or "lobster: not found"

**Symptom:** Running `$LOBSTER` returns error.

**Cause:** The alias isn't set, or the path is wrong.

**Fix:**
```bash
# Check if the file exists
ls -la ~/clawd/tools/lobster/bin/lobster.js

# Set the alias with correct path
LOBSTER="node /path/to/lobster/bin/lobster.js"

# Or install globally
npm install -g @clawdbot/lobster
```

### State directory permission errors

**Symptom:** `EACCES: permission denied` when running pipelines with `diff.last` or `state.set`.

**Cause:** Lobster can't write to `~/.lobster/state/`.

**Fix:**
```bash
# Create directory with correct permissions
mkdir -p ~/.lobster/state
chmod 755 ~/.lobster/state

# Or override with writable location
export LOBSTER_STATE_DIR=/tmp/lobster-state
```

### Approval token expired or invalid

**Symptom:** `resume` fails with "invalid token" or "token expired".

**Cause:** Approval tokens expire after 24 hours by default. Or the token was corrupted during copy/paste.

**Fix:**
```bash
# Re-run the original pipeline to get a fresh token
$LOBSTER run --mode tool 'your-pipeline-here'

# Make sure token is quoted properly (contains special chars)
$LOBSTER resume --token "eyJ..." --approve yes
```

**Prevention:** Process approvals promptly. Don't let them sit overnight.

### JSON parsing errors

**Symptom:** `Unexpected token` or `JSON parse error` in pipeline output.

**Cause:** A command in the pipeline returned non-JSON output, or the JSON is malformed.

**Fix:**
```bash
# Debug: run each step separately
$LOBSTER 'exec --json --shell "your-command"' 
# Check: is stdout valid JSON?

# Common issue: command outputs extra text before JSON
# Fix: use jq or grep to extract JSON portion
$LOBSTER 'exec --json --shell "your-command | tail -1"'
```

**Tip:** Use `--json` flag with `exec` to enforce JSON parsing. Without it, output is treated as plain text.

### Pipeline hangs or times out

**Symptom:** Pipeline doesn't return, or returns after long delay.

**Cause:** Usually a shell command inside `exec` is hanging (waiting for input, network timeout, etc.).

**Fix:**
```bash
# Add timeout to shell commands
$LOBSTER 'exec --json --shell "timeout 30 your-command"'

# Check if command works standalone
your-command  # Does it hang by itself?

# For network commands, check connectivity
curl -I https://api.github.com
```

### diff.last always returns all items

**Symptom:** `diff.last` never filters anything, always shows full list.

**Cause:** The `--key` is different each run, or state was cleared.

**Fix:**
```bash
# Use consistent keys
diff.last --key "my-workflow:consistent-name"  # Good
diff.last --key "my-workflow:$(date)"          # Bad - different every time

# Check state directory
ls ~/.lobster/state/

# Clear state to reset (if needed)
rm ~/.lobster/state/my-key.json
```

### clawd.invoke fails with connection error

**Symptom:** `ECONNREFUSED` or "Clawdbot not available".

**Cause:** Environment variables not set, or Clawdbot gateway not running.

**Fix:**
```bash
# Check required env vars
echo $CLAWD_URL   # Should be http://localhost:PORT
echo $CLAWD_TOKEN # Should be set

# Check if gateway is running
curl $CLAWD_URL/health

# Restart gateway if needed
openclaw gateway restart
```

---

## State Directory

Lobster stores state in `~/.lobster/state/` by default. Override with `LOBSTER_STATE_DIR`.
