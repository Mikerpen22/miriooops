---
author:
  name: "Minhung Shih"
date: 2026-04-17
linktitle: "Auto-Harness: Frankensteining my way to a better agent"
title: "Auto-Harness: Frankensteining my way to a better agent"
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

If you've been living inside Claude Code for more than a couple of weeks, you probably have the same problem I do: a drawer full of harnesses.

There's `superpowers` over here doing its serious "brainstorm → plan → execute → review" dance. There's `ralph-loop` over there, happily running a tight autonomous loop until the CI turns green. There's `pr-review-toolkit` with its specialized reviewer subagents. Each one is good. Each one has opinions.

The trouble starts the moment I try to do real work.

> "I want the brainstorming discipline from superpowers, but the autonomous loop from ralph, but with a judge agent at the end like the PR toolkit does. And please don't make me copy-paste three SKILL.md files together at 11pm."

That sentence, in some awkward variation, is why I built `auto-harness`.

### What I actually wanted

Not another harness. I already had harnesses. I wanted a **meta-harness**. Something that could look at my project, look at what's already installed, and spit out a skill that borrows the right bits from the right places.

Three properties I refused to compromise on:

1. **It had to run entirely inside Claude Code.** No Python service, no API orchestrator, no "just spin up this FastAPI." If it needs a runtime, it's dead on arrival.
2. **It had to have a token budget.** Meta-anything burns tokens. If it's not bounded, it's a footgun.
3. **It had to be honest about what it doesn't know.** A generated harness is a guess, not a benchmark result. I wanted the output to say so, out loud.

### The shape it took

`auto-harness` runs a seven-phase flow. The interesting part isn't the phases. It's the gate between Phase 2 and Phase 3:

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

Phase 3 is the expensive phase. Parallel subagent dispatch is how you get real variation instead of one idea in a trench coat. It's also where your token bill explodes. So the skill stops and asks:

```
I'm about to spawn 4 variant-builder subagents in parallel.
This is the token-expensive step. Proceed? (y/N)
```

Small detail. Saves me from myself all the time.

### The catalog trick

Rather than letting the model invent a harness shape from scratch, I gave it a short, opinionated catalog of six patterns:

- `three-agent`: Planner → Generator → Evaluator (long coding tasks)
- `research-triad`: Searcher → Synthesizer → Verifier (fact-grounded writing)
- `single-with-critic`: one agent + one critic pass (short, bounded work)
- `pipeline`: deterministic stages (ETL-shaped tasks)
- `ralph-style-loop`: single-agent autonomous loop (open-ended)
- `superpowers-workflow`: brainstorm → plan → execute → review

Each pattern has a **probe file**: a list of the failure modes that pattern tends to hit. The variant-builders have to answer them. The judge scores against them.

> Opinionated beats generative here. Six patterns with sharp edges are more useful than infinite patterns with dull ones.

There's a free-form escape hatch when nothing fits. It's used less than you'd think.

### The compose / extend / replace menu

When `auto-harness` scans `~/.claude/` and finds, say, `superpowers` installed, it doesn't barrel past it. It asks:

```
Detected: superpowers, ralph-loop, pr-review-toolkit

How should the generated harness relate to what you have?

  standalone  : fully self-contained (default)
  compose     : delegate specific phases to installed skills
  extend      : wrap or inherit from an installed harness
  replace     : domain-specialized alternative to an installed harness
```

`compose` is the one I use most. It's how you get "brainstorming from superpowers, autonomous loop from ralph, judge from the PR toolkit" declared explicitly, in one skill, instead of a fragile manual stitch.

### The honest scorecard

This is the part I'm most stubborn about. Every generated bundle ships with a `scorecard.md` that looks roughly like this:

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

The moment you let "estimated" quietly become "measured" in a reader's head, you've lost the plot. So the disclaimer stays loud, and every bundle also includes a fillable `evaluation-protocol.md`, the template for when you actually want to measure the thing.

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

Answer the two intake questions. Pick a composition strategy. Approve the parallel dispatch. Forty-ish seconds later you have a `./generated-skill/` directory with a SKILL.md you can actually use.

### What it's not

- **Not a benchmark.** The scorecard is a guess. Real data comes from running the generated skill on your real work.
- **Not a universal harness.** Six patterns plus an escape hatch is opinionated on purpose.
- **Not a "one-click agent factory."** It's a tool for people who already have harness opinions and want to stop copy-pasting.

### What I actually learned building it

1. **The most useful "skill" is often one that orchestrates other skills.** My setup got a lot more useful the moment I stopped treating each harness as a closed world.
2. **Bounded budgets make meta-tools usable.** A hard cap of 9 subagent dispatches is what keeps this from being a novelty demo. Without it, I'd never run it twice.
3. **Parallel subagents produce different ideas only when you force the mutation.** Each variant-builder gets a distinct 2 to 3 knob mutation set. Without that, you get four slightly-reworded copies of the same harness. With it, you get real divergence, and the judge has something to judge.
4. **The boring part is what separates a toy from a tool.** The bundle schema, the frontmatter linter, the "write to disk last" gate. None of it is exciting. All of it matters.

### Links

- **Repo:** [github.com/Mikerpen22/auto-harness](https://github.com/Mikerpen22/auto-harness)
- **License:** MIT
- **Contributing:** PRs for new patterns or probes are welcome. Probe PRs need a written justification of the failure mode the probe catches. (Yes, that's on purpose.)

If you try it and the generated harness is wrong for your project, tell me how. That's more useful than a star, honestly.
