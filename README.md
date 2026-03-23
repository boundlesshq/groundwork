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
/plugin marketplace add boundlesshq/groundwork
```

### Install the plugin

```
/plugin install groundwork@boundlesshq-groundwork
```

## Idea behind groundwork

The idea behind this skill suite is that sometimes you want to use Claude as a soundboard rather than a "task do-er".

I was hitting a wall where I'd want to do a codebase exploration for a ticket, and Claude would be like "I know what to do here" and write the code or whip up a plan before I was done understanding the problem. I'd let it do its thing, and more often than not, it would 'work' but it wouldn't be the correct approach. Sometimes that's my fault (I didn't fully understand the problem when I originally prompted it) and other times it was just a matter of complexity (and fragile tightly coupled things that I didn't know about).

### Inspiration

I took inspiration from the very cool, very popular [superpowers](https://github.com/obra/superpowers) skill. Basically, superpowers does a proper brainstorm session with you, and has an interactive session where you decide on implementation, architecture, etc. Then it hands that plan off to a plan writer, and then it orchestrates a group of sub-agents who go work on the feature using TDD.

My favorite part was the brainstorming session -- but I wanted one that was more of a "let's think about this together" rather than one that put a bunch of ideas in my head before I had the chance to actually think the problem through myself.

Also, I can sometimes have a hard time staying focused while working on tickets and wanted to be able to ask exploratory questions, but not go down every rabbit hole I saw when I wanted to know how something was working.

### The flow

1. **ticket-analysis** looks at the Jira ticket together using Atlassian MCP and makes sure A/C is all clear. If not, pause while we get clarification from the team or product.

2. Once all the A/C is agreed upon and no open questions remain, pass off to the **architect** who helps plan the implementation. This session is also interactive because I want to completely understand the approach before we write code. This step sounds the most straightforward but also feels like it needs the most tuning to get right. It's not working as I envision it still, but we'll get there.

3. Finally, the architect writes up a plan for the **pairing-buddy** who will be my rubber-duck that keeps me unblocked while I work on the ticket. I write the code. Pairing-buddy can only give me pseudocode if I'm blocked (ideally... more on that later) -- this lets me stay sharp and do the fun part about programming: writing code. If I'm really stuck I can ask for code snippet examples, but so far I haven't really needed/wanted that yet. Pairing-buddy is also supposed to help me stay focused. I can ask questions about stuff I'm curious about while working, but if I go too far down a rabbit hole, it's supposed to pull me back.

### The point

It's really supposed to be all about not robbing me of my chances to grow as a dev, and letting me think about problems. It's not a skill designed to be a productivity multiplier, but more one that promotes critical thinking and not pressing the magical " ✨ get work done ✨ " button (which can often lead to longer turnaround time as I may not have thought the problem through very well).

### Where it's at right now

Groundwork doesn't meet my vision yet. Some things I'm working through:

- **Pairing-buddy was getting spoiled.** The architect was passing full implementation instructions, and since pairing-buddy's entire existence is the prompt and its context, it just told me solutions (spoiling all my fun). My workaround was to have the architect make 2 plans -- one with instructions to give me, and another with instructions that are more "machine friendly".
- **No seamless handoffs yet.** The skills don't hand off to one another using hooks, which is just down to me not yet taking the time to learn how to implement hooks into a skill.

I plan on working on the skill suite more soon, but haven't had the time to tinker with it. Since we all started discussing skills lately, figured I'd share where it's at.

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
