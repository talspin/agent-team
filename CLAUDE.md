# Agent Team

This repo defines a reusable dev team of Claude Code subagents with well-defined roles and handoff contracts.

## Team Structure

```
PM → Architect → Implementer(s) → Reviewer → QA
```

Each agent has a narrow job and produces structured output consumed by the next agent.

## Handoff Contracts

Every agent outputs a structured block at the end of their response. Do not skip this — it is the input contract for the next agent.

### PM output
```
## Spec
- title:
- acceptance_criteria: []
- out_of_scope: []
- open_questions: []
```

### Architect output
```
## Plan
- files_to_modify: []
- new_files: []
- risks: []
- estimated_complexity: low | medium | high
```

### Implementer output
```
## Implementation Summary
- branch:
- tests_passing: true | false
- self_review_notes: []
```

### Reviewer output
```
## Review
- verdict: approve | request_changes | block
- findings: []  # each: { severity, file, line, issue, suggestion }
```

### QA output
```
## QA Report
- verdict: pass | fail
- tests_added: []
- failures: []  # each: { test, error, suggestion }
```

## Quality Gates

- Architect plan must be approved by a human before Implementer starts
- Reviewer `block` verdict returns to Architect (design issue), not Implementer
- Reviewer `request_changes` returns to Implementer
- QA `fail` returns to Implementer with the full failure report
- No PR merges unless Reviewer approves AND QA passes

## Conventions

- Use `isolation: worktree` for any agent that writes files
- Orchestrator uses Opus; workers use Sonnet
- Keep agent prompts narrow — each agent should do one thing well
- If a task spans multiple layers (frontend/backend/tests), spawn parallel Implementers with non-overlapping file ownership
