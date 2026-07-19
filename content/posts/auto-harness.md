---
author:
  name: "Minhung Shih"
date: 2026-04-17
linktitle: "Auto-Harness: Frankensteining my way to a better agent"
title: "Auto-Harness: Frankensteining my way to a better agent"
description: "How I built a Claude Code skill that inspects the harnesses I already use, generates several designs, and assembles the best parts into a new SKILL.md."
type:
- post
- posts
weight: 10
series:
- AI Tooling
tags:
- claude-code
- agents
- skills
- meta
aliases:
---

### The harness drawer is a mess

After using Claude Code for a few weeks, I ended up with a collection of agent harnesses that did not fit together.

`superpowers` provides a structured brainstorm → plan → execute → review workflow. `ralph-loop` runs an autonomous loop until the CI passes. `pr-review-toolkit` uses specialized reviewer subagents. Each is useful, but each assumes that its own workflow controls the task.

My projects rarely line up that neatly. I might want `superpowers` to plan the work, a Ralph-style loop to grind through it, and a separate reviewer at the end. After manually splicing those instructions together a few times, I built `auto-harness` to do the splicing for me.

### What I needed

Writing a fourth fixed workflow would only make the drawer messier. I wanted a **meta-harness**: something that could inspect the current project and the skills already installed, then build a purpose-specific skill from the useful pieces.

I also wanted it to remain boring to operate. It runs inside Claude Code, with no Python service or API orchestrator on the side. It puts a hard ceiling on subagent calls because generating several candidates gets expensive quickly. And whenever a model judges another model's work, the result is labeled as an opinion rather than dressed up as benchmark data.

### The shape it took

`auto-harness` uses seven phases:

```
Phase 1:    Intake              : restate the project, synthesize a scenario
Phase 1.5:  Environment recon   : scan ~/.claude/ for installed harnesses
Phase 2:    Route               : pick base pattern(s) from a catalog of 6
Phase 3:    Parallel variants   : dispatch 3 to 5 variant-builders in parallel
Phase 4:    Judge scoring       : a judge subagent ranks them on a rubric
Phase 5:    Optional mutation   : one round of focused refinement
Phase 6:    Synthesize champion : build the final SKILL.md bundle
Phase 7:    Emit & install      : write to ./generated-skill/ (or ~/.claude/skills/)
```

Phase 3 spends most of the tokens. Before it launches the variant builders, the skill stops and says exactly what it is about to do:

```
I'm about to spawn 4 variant-builder subagents in parallel.
This is the token-expensive step. Proceed? (y/N)
```

I have said no to that prompt more often than I expected.

### Starting from known patterns

Starting from a blank prompt produced workflows that sounded different but behaved almost identically. The fix was a small catalog of structures with known tradeoffs:

- `three-agent`: Planner → Generator → Evaluator (long coding tasks)
- `research-triad`: Searcher → Synthesizer → Verifier (fact-grounded writing)
- `single-with-critic`: one agent + one critic pass (short, bounded work)
- `pipeline`: deterministic stages (ETL-shaped tasks)
- `ralph-style-loop`: single-agent autonomous loop (open-ended)
- `superpowers-workflow`: brainstorm → plan → execute → review

Each pattern has a **probe file** describing the failures it tends to invite. A long autonomous loop can lose the original goal; a research pipeline can pass the same unsupported claim from one agent to the next. The variant builders have to account for those failures, and the judge scores how well they did.

Those six patterns have covered my projects so far. The free-form route is there for the odd case that does not fit.

### The compose / extend / replace menu

When `auto-harness` finds installed skills such as `superpowers`, it asks how the generated skill should use them:

```
Detected: superpowers, ralph-loop, pr-review-toolkit

How should the generated harness relate to what you have?

  standalone  : fully self-contained (default)
  compose     : delegate specific phases to installed skills
  extend      : wrap or inherit from an installed harness
  replace     : domain-specialized alternative to an installed harness
```

I use `compose` most often. The generated skill records which installed skill owns each phase, so I can see the handoffs without digging through copied instructions.

### Scores are estimates, not benchmarks

Every generated bundle includes a `scorecard.md`:

```markdown
| Variant | Clarity | Recovery | Fit | Total |
|---------|---------|----------|-----|-------|
| v1 (baseline)        | 7 | 6 | 8 | 21 |
| v2 (+strict judge)   | 6 | 8 | 7 | 21 |
| v3 (+blended router) | 8 | 7 | 9 | 24 ← champion
| v4 (+compact loop)   | 7 | 7 | 7 | 21 |

> Note: scores are estimated by a judge subagent, not measured.
> Run `evaluation-protocol.md` against your real tasks to get real data.
```

The numbers are only the judge's shorthand for its preference. They help choose which candidate to inspect first; they do not show that the candidate works. Each bundle therefore includes an `evaluation-protocol.md` for running the generated skill against real tasks.

### Using it

```bash
git clone https://github.com/Mikerpen22/auto-harness.git
cd auto-harness
./install.sh
```

Then, inside any Claude Code session:

```
/auto-harness

> I want a coding agent that refactors legacy Django views.
```

Answer the two intake questions, choose a composition strategy, and approve the parallel dispatch. On my setup, a typical run takes about 40 seconds and writes the result to `./generated-skill/`.

### What it cannot decide

`auto-harness` can compare candidate workflows, but it cannot decide what "good" means for a project. A refactoring agent may need caution and clean checkpoints; a migration agent may need persistence and a strong rollback path. The intake questions capture some of that context, but the generated `SKILL.md` still needs to be read by the person who will run it.

I keep the catalog narrow for the same reason. I would rather inspect a recognizable workflow than untangle a novel agent hierarchy after it fails.

### What changed while building it

My first version simply asked several subagents for their best design. They returned the same workflow in different prose. Parallelism alone did not create useful variety.

Now each variant builder gets a different set of two or three changes to explore: stricter recovery, a smaller loop, a separate judge, a different routing rule. Once the candidates are forced to disagree, the comparison becomes useful.

The token limit also moved from a suggestion to a rule. `auto-harness` allows at most nine subagent dispatches. If the budget lived only in the prompt, every phase could find a good reason to spend a little more.

The part I trust most is not the judge. It is the bundle schema that catches missing files, the front matter linter that catches skills Claude Code cannot load, and the "write to disk last" rule that keeps an interrupted run from leaving behind something that looks installable. Those checks are what make me comfortable using the generated skill.

### Links

- **Repo:** [github.com/Mikerpen22/auto-harness](https://github.com/Mikerpen22/auto-harness)
- **License:** MIT
- **Contributing:** PRs for new patterns or probes are welcome. A new probe should explain the failure it is meant to catch.

If a generated harness falls apart on your project, open an issue with the failure case. That is the part I want to learn from.
