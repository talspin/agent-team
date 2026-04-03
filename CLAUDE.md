# Agent Team

This repo defines a reusable Claude Code subagent dev team with well-defined roles and handoff contracts.

## Team

| Agent | Model | Owns |
|---|---|---|
| `orchestrator` | Opus | Full pipeline coordination, GitHub issue lifecycle, agent communication log |
| `pm` | Sonnet | Specs and acceptance criteria |
| `architect` | Opus | Implementation plans |
| `git-ops` | Sonnet | Branch creation, rebase, conflict resolution, PR merging, worktree cleanup |
| `implementer` | Sonnet | Code (worktree-isolated), local validation (unit/integration/e2e/prober/docker) |
| `reviewer` | Opus | Code correctness, security, conventions |
| `sre` | Opus | Production safety, infra, runbooks, deployment config, prod validation |
| `qa` | Sonnet | Tests, CI/CD pipeline config, PR CI check status (`gh pr checks`) |
| `tech-writer` | Sonnet | Everything under `docs/` |

## Pipeline

```
Feature Request
    → pm          → Spec                      (human reviews open_questions)
    → architect   → Plan                      ← HUMAN APPROVAL GATE
    → git-ops     → branch created
    → implementer → Code + tests
    → git-ops     → rebase + check-ready      (before PR)
    → reviewer    → approve | request_changes | block
    → sre         → prod-safe | needs-changes | block
    → qa          → pass | fail               (tests + CI/CD + PR checks)
    → tech-writer → docs updated
    → git-ops     → merge PR                  (auto-merge after all checks pass)
    → git-ops     → cleanup
    → DONE
```

## Feedback loops

| Verdict | From | Route to |
|---|---|---|
| `block` | reviewer | architect |
| `request_changes` | reviewer | implementer |
| `block` | sre | architect |
| `needs-changes` | sre | implementer |
| `fail` | qa | implementer |

## Handoff Contracts

Every agent outputs a structured block at the end of their response. This is the input contract for the next agent.

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
- interfaces: []
- risks: []
- tests_to_update: []
- new_tests_needed: []
- estimated_complexity: low | medium | high
```

### Implementer output
```
## Implementation Summary
- branch:
- files_changed: []
- tests_passing: true | false
- self_review_notes: []
- deviations_from_plan: []
```

### Reviewer output
```
## Review
- verdict: approve | request_changes | block
- summary:
- findings: []  # each: { severity, file, line, issue, suggestion }
```

### SRE output
```
## SRE Review
- verdict: prod-safe | needs-changes | block
- summary:
- deployment_notes:
- findings: []  # each: { severity, area, file, issue, blast_radius, suggestion }
- runbooks_updated: []
```

### QA output
```
## QA Report
- verdict: pass | fail
- tests_added: []
- suite_results: { total, passing, failing }
- ci_pipeline_changes: []
- ci_status: passing | failing | not_run
- failures: []  # each: { test, file, error, likely_cause, suggestion }
```

### Tech-writer output
```
## Docs Update
- docs_created: []
- docs_updated: []
- changelog_updated: true | false
- gaps_found: []
```

## Quality Gates

- Architect plan must be approved by a human before Implementer starts
- Reviewer `block` → back to Architect; `request_changes` → back to Implementer
- SRE `block` → back to Architect; `needs-changes` → back to Implementer
- QA `fail` → back to Implementer
- git-ops auto-merges the PR after Reviewer approves AND SRE approves AND QA passes AND docs are updated — no human approval required for merge
- Docs must be updated before the PR is merged

## Conventions

- `isolation: worktree` on any agent that writes files (implementer, qa, tech-writer)
- Orchestrator and reviewers (reviewer, sre, architect) use Opus; workers use Sonnet
- Parallel Implementers are allowed for changes spanning independent layers — assign non-overlapping file ownership
- If a feedback loop iterates 3+ times on the same issue, escalate to the human
