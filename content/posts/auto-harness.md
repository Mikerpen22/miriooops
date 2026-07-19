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

My actual projects often need parts of all three:

> "I want the brainstorming discipline from superpowers, but the autonomous loop from ralph, but with a judge agent at the end like the PR toolkit does. And please don't make me copy-paste three SKILL.md files together at 11pm."

I built `auto-harness` to assemble that combination without manually merging several `SKILL.md` files.

### What I needed

I did not want another fixed workflow. I wanted a **meta-harness** that could inspect the project and the installed skills, then generate a new skill from the relevant parts.

Three properties I refused to compromise on:

1. **It had to run entirely inside Claude Code.** I did not want a separate Python service or API orchestrator to install and maintain.
2. **It had to enforce a token budget.** Generating and evaluating several variants is expensive, so the number of subagent calls must be bounded.
3. **It had to distinguish estimates from measurements.** A judge subagent can rank generated variants, but that ranking is not a benchmark.

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

Phase 3 is the expensive part. Running several subagents can produce meaningfully different designs, but it also consumes most of the token budget. Before starting that phase, the skill asks for confirmation:

```
I'm about to spawn 4 variant-builder subagents in parallel.
This is the token-expensive step. Proceed? (y/N)
```

That prompt has prevented more than one accidental, expensive run.

### Starting from known patterns

Instead of asking the model to invent every workflow from scratch, I gave it a catalog of six patterns:

- `three-agent`: Planner → Generator → Evaluator (long coding tasks)
- `research-triad`: Searcher → Synthesizer → Verifier (fact-grounded writing)
- `single-with-critic`: one agent + one critic pass (short, bounded work)
- `pipeline`: deterministic stages (ETL-shaped tasks)
- `ralph-style-loop`: single-agent autonomous loop (open-ended)
- `superpowers-workflow`: brainstorm → plan → execute → review

Each pattern has a **probe file** listing the failure modes it is likely to encounter. Variant builders must address those failure modes, and the judge includes them in its scoring.

The catalog makes the output more predictable and easier to review. A free-form option remains available when none of the six patterns fits.

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

I use `compose` most often. It records which installed skill handles each phase, so the relationship is explicit and does not depend on manually copied instructions.

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

The score is explicitly labeled as an estimate. Each bundle also includes an `evaluation-protocol.md` template for testing the generated skill on real tasks. This prevents a model-generated ranking from being presented as measured performance.

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

### What it's not

- **Not a benchmark.** The scorecard is a guess. Real data comes from running the generated skill on your real work.
- **Not a universal harness.** Six patterns plus an escape hatch is opinionated on purpose.
- **Not a "one-click agent factory."** It assumes that the user can review the generated workflow and knows what tradeoffs matter for the project.

### Lessons from building it

1. **Orchestration can be more useful than another standalone skill.** My setup improved once the harnesses could delegate to one another instead of each trying to own the entire task.
2. **A budget has to be enforced in code or instructions.** `auto-harness` allows at most nine subagent dispatches. Without that limit, I would hesitate to run it on ordinary work.
3. **Parallel subagents need distinct assignments.** Each variant builder receives a different set of two or three design changes. Without those constraints, the outputs differ mostly in wording. With them, the judge has genuinely different designs to compare.
4. **The supporting checks make the output usable.** The bundle schema, front matter linter, and "write to disk last" rule are less visible than the generation step, but they prevent malformed or partial skills from being installed.

### Links

- **Repo:** [github.com/Mikerpen22/auto-harness](https://github.com/Mikerpen22/auto-harness)
- **License:** MIT
- **Contributing:** PRs for new patterns or probes are welcome. Probe PRs need a written justification of the failure mode the probe catches. (Yes, that's on purpose.)

If a generated harness does not fit your project, open an issue with the failure case. That feedback is more useful to me than a star count.
