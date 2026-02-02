# Lobster Vision

> Typed workflow runtime with approval gates for AI agents

## Purpose

Lobster exists to give AI agents **predictable, efficient automation** without sacrificing human oversight.

The core insight: Most agent workflows are repetitive pipelines that don't need re-planning on each run. An email triage workflow is always: fetch → filter → approve → act. A PR monitor always: fetch → diff → notify. Re-planning these pipelines burns tokens and introduces non-determinism.

Lobster solves this by letting agents **define pipelines once**, then execute them deterministically with:
- **Typed JSON data flow** between steps
- **Approval gates** that halt and resume on human decision
- **Stateful diffing** to detect changes across runs

## Philosophy

**1. Determinism over flexibility**
Pipelines should produce the same output given the same input. No LLM inference mid-pipeline. Side effects are explicit and gated.

**2. Human oversight built-in**
The `approve` command is first-class. Workflows that modify data or send messages should halt and ask. Agents should never "just do it" with side effects.

**3. Token efficiency**
Running a 10-step pipeline as one Lobster command costs one tool call. Running it as 10 agent decisions costs 10× the tokens plus unpredictable reasoning.

**4. Composability**
Small commands compose into pipelines. Pipelines compose into workflows. Workflows can call Clawdbot tools via `clawd.invoke`.

## Target Use Cases

### Primary
- **Email triage** — Fetch unread, filter by criteria, approve actions, mark processed
- **PR/issue monitoring** — Detect changes, notify only when state changes
- **Batch operations** — Process lists with approval before each action
- **Approval workflows** — Any multi-step task where a human should verify before side effects

### Secondary
- **Data transformation** — ETL-style pipelines for JSON data
- **Scheduled checks** — Heartbeat-triggered monitoring via cron
- **Report generation** — Aggregate data and format for output

## Non-Goals

Lobster is **not**:

- **A general scripting language** — Use bash/Python for complex logic. Lobster is for pipelines, not programs.
- **A replacement for orchestration tools** — Kubernetes, Temporal, Airflow handle distributed systems. Lobster handles agent-local workflows.
- **An LLM reasoning engine** — No inference happens inside pipelines. Lobster executes; agents reason.
- **A database** — State is simple key-value. Complex querying needs external tools.

## Roadmap Priorities

### Phase 1: Stability (Current)
- Solidify core commands (`exec`, `where`, `approve`, `diff.last`)
- Documentation and examples
- Error messages that help debug

### Phase 2: Integration
- First-class Clawdbot tool integration (`clawd.invoke`)
- Webhook triggers for event-driven workflows
- Better workflow file format

### Phase 3: Ecosystem
- MoltHub skill publishing
- Community workflow library
- Visual pipeline builder (maybe)

## Success Metric

Lobster succeeds when an agent can say:
> "I have a workflow for that" → runs one command → gets approval → completes task

Instead of:
> "Let me think about how to do this..." → 10 tool calls → maybe works → burns tokens
