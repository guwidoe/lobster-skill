# Proposal: Foundation Documentation

**Date:** 2025-02-02  
**Scope:** 2-4 hours  
**Focus Area:** Documentation (per ACTIVE_REPOS.md)

## Problem

The lobster-skill repository is missing foundational documentation that enables effective development and contribution:

1. **No VISION.md** — Contributors don't know the project's direction
2. **No practical examples** — SKILL.md mentions email triage but doesn't show how
3. **Limited troubleshooting** — Users hit errors without guidance

## Proposed Features

### 1. Add VISION.md (30 min)

Create a clear vision document covering:
- Project purpose and philosophy
- Target use cases
- Non-goals (what Lobster is NOT)
- Roadmap priorities

**Deliverable:** `VISION.md` in repo root

### 2. Add Email Triage Workflow Example (2 hours)

The SKILL.md says Lobster is for "Email triage or batch operations" but provides no email example. Add:

- A complete email triage workflow example in SKILL.md
- Shows: fetch unread → filter by sender → approve → mark read
- Demonstrates approval gates + stateful diff.last

**Deliverable:** New section in `SKILL.md` with working email pipeline example

### 3. Add Troubleshooting Section (1 hour)

Document common issues:
- "Command not found" (path issues)
- State directory permissions
- Approval token expiry
- JSON parsing errors

**Deliverable:** New "Troubleshooting" section in `SKILL.md`

## Success Criteria

- [ ] VISION.md exists and clearly defines project direction
- [ ] SKILL.md includes email triage example that agents can copy/adapt
- [ ] Troubleshooting section covers 4+ common issues
- [ ] All changes committed and pushed

## Why This Matters

1. **VISION.md** enables future development cycles (required for orchestrator workflow)
2. **Practical examples** help agents actually use Lobster (the whole point!)
3. **Troubleshooting** reduces support burden and improves agent autonomy

## Implementation Notes

- Keep examples copy-paste ready
- Use realistic tool names (e.g., actual Clawdbot message/email tools)
- Test examples mentally against SKILL.md capabilities
