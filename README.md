# Agent Team

A reusable Claude Code subagent team for shipping features with high quality.

## Roles

| Agent | Model | Job |
|---|---|---|
| `pm` | Sonnet | Turns a feature request into a structured spec |
| `architect` | Opus | Turns a spec into a concrete implementation plan |
| `implementer` | Sonnet | Executes the plan, writes code, runs tests |
| `reviewer` | Opus | Reviews code for bugs, security, and correctness |
| `qa` | Sonnet | Fills test gaps and runs the full suite |

## Pipeline

```
Feature Request
    → pm          → Spec
    → architect   → Implementation Plan  ← human approval gate
    → implementer → Code + passing tests
    → reviewer    → approve | request_changes | block
    → qa          → pass | fail
    → Merge
```

Feedback loops:
- Reviewer `block` → back to Architect
- Reviewer `request_changes` → back to Implementer
- QA `fail` → back to Implementer

## Usage

### In a project (git submodule)

```bash
git submodule add <this-repo-url> .agent-team
ln -s ../.agent-team/.claude/agents .claude/agents
```

### In a project (copy)

Copy `.claude/agents/` into your project's `.claude/agents/` directory.

### Invoking agents

In Claude Code, agents are invoked automatically when you describe a task that matches their `description` field. You can also invoke explicitly:

```
use the pm agent to write a spec for: <feature request>
use the architect agent to plan this spec: <spec>
use the implementer agent to implement this plan: <plan>
use the reviewer agent to review the changes on branch <branch>
use the qa agent to test the implementation
```

## Customizing for your project

Add project-specific context to `CLAUDE.md` in your project root — all agents inherit it automatically. Use individual agent files only for role-specific behavior.

To experiment with variations, fork this repo and modify agent prompts. Each agent is a single Markdown file — easy to diff and iterate.
