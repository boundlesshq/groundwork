# groundwork

A Claude Code plugin providing a structured feature development workflow. Four skills that chain together from ticket to PR.

## Workflow

```
ticket-analysis → architect → pairing-buddy → local-review
```

1. **ticket-analysis** — Walk through a Jira ticket AC-by-AC to lock down requirements before writing any code
2. **architect** — Brainstorm approaches, compare tradeoffs, agree on a design, and write an implementation plan to disk
3. **pairing-buddy** — Navigate the developer through implementation step-by-step, managing rabbit holes and keeping focus
4. **local-review** — Pre-PR correctness check: bugs, missing coverage, AC gaps — no style opinions

Each skill works standalone in a fresh session. Running them in sequence in the same session passes context forward automatically.

## Install

### Add the marketplace

```
/plugin marketplace add rcopra/groundwork
```

### Install the plugin

```
/plugin install groundwork@rcopra-groundwork
```

### Optional: prompt your team automatically

Add to your project's `.claude/settings.json` to prompt teammates on first open:

```json
{
  "extraKnownMarketplaces": {
    "rcopra-groundwork": {
      "source": {
        "source": "github",
        "repo": "rcopra/groundwork"
      }
    }
  },
  "enabledPlugins": {
    "groundwork@rcopra-groundwork": true
  }
}
```

## Requirements

- **ticket-analysis** requires the Jira MCP server connected to Claude Code
- All other skills work with no extra setup

## Skills

### ticket-analysis

Fetches the ticket and walks through each AC interactively using `AskUserQuestion`. Ensures requirements are unambiguous before any design work begins. Will not speculate on implementation — that's the architect's job.

**Triggers:** "analyze this ticket", "let's look at PROJ-123", "check the AC", "what are we building"

### architect

Phased architecture session: establish constraints → explore the codebase with subagents → compare 2+ approaches with honest tradeoffs → agree on a design → write an implementation plan to `.claude/plans/`.

No generated code — pseudocode only (English sentences).

**Triggers:** "how should I build this", "let's plan the architecture", "architect this feature"

### pairing-buddy

Navigator while the developer drives implementation. Tracks plan steps, handles three levels of rabbit holes (productive / adjacent / off-track), and helps the developer discover the "why" without spoon-feeding. Reads the plan from `.claude/plans/`.

**Triggers:** "let's start coding", "pair with me", "let's pair", "open the plan"

### local-review

Reads `git diff main...HEAD` and the plan file. Reports Issues (fix before PR) and Notes (informational) only. Will not suggest renames, refactors, or improvements beyond what the ticket asked for.

**Triggers:** "review my code", "check my work", "ready for review", "pre-PR check"
