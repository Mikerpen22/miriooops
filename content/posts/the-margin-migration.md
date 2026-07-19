---
author:
  name: "Minhung Shih"
date: 2026-07-18
linktitle: "The Margin Migration"
title: "The Margin Migration: what Kimi K3 actually repriced"
description: "An open-weight model landed three points off the frontier at a fraction of the price. The interesting part isn't that open source is catching up. It's where the margin goes when the model layer stops being the scarce thing."
image: ""
type:
- post
- posts
weight: 10
series:
- Markets
tags:
- ai-infrastructure
- llm
- hardware
- open-source
- markets
aliases:
---

I spend most of my day thinking about systems: where the bottleneck is, who pays for it, and what happens when you move it. Lately the most interesting system I can find isn't in a codebase. It's the stack of companies that turns electricity into tokens.

Something shifted in that stack over three days in July, and I don't think it's priced yet. This is me writing to find out whether I actually believe it.

A note before we start: none of this is investment advice, and my confidence is uneven. I've tried to be honest about which claims survived a real fact-check and which are one analyst's framing I happen to find plausible. There's a section at the end that spells out how I sorted them.

### Seventy-two hours in July

On July 16, Moonshot AI shipped **Kimi K3**: a 2.8-trillion-parameter sparse mixture-of-experts model that activates 16 of its 896 experts per token, runs a million-token context, and ships open weights.[^k3] On Artificial Analysis's intelligence index it lands around 57. Claude Fable 5 sits at ~60, GPT-5.6 Sol at ~59.[^aa] So an open-weight model from a Chinese lab is now roughly three index points off the best closed models in the world, at open-weight pricing.

The next day, Gavin Baker, a tech investor whose posts are about as close as you get to watching Wall Street form a consensus in real time, wrote the read that stuck:

> "Kimi K3 may be an important inflection point for AI. Potentially negative for Anthropic and OpenAI while being net positive for essentially every other company in the world... the real Sputnik moment would be an open-source frontier model that was also token efficient."
>
> — Gavin Baker, X, July 17 2026[^baker]

That last clause is the whole essay in one sentence, and almost every retelling drops it. Hold onto it.

Here's the reframe I want to argue for. The trade everyone reaches for is "open source wins, closed labs lose." That's too blunt, and I think it's slightly wrong. The thing that actually happened is subtler and more durable:

> The model layer's margin didn't disappear. It migrated one floor down.

Let me build that up from first principles, then show you why it's really a hardware story wearing a markets costume.

### The monopsony inversion

Decompose the price of an AI token and you get, roughly: **power + silicon + data center + capital + model margin.**

For the last two years we've lived in a world of two or three dominant closed labs. In that world the labs are monopsonies over everything beneath them. They're the buyer that sets the terms for chips, for power, for capacity. And the model layer, the one layer with genuine software-like margins, kept a fat slice of every token dollar because nobody could undercut it on quality.

Open-weight parity attacks exactly that slice. When a model three points off the frontier is free to download, competition arrives at the one floor that had pricing power, and the margin it loses has to reappear somewhere. It reappears as demand and margin one floor down: the clouds that serve the model, the silicon that runs it, the memory and power and interconnect that stay supply-constrained no matter whose weights are loaded.

The token gets cheaper. The token count explodes. And the toll collectors below the model keep their pricing power, because *their* floors are still the scarce ones.

That's the migration. It's not "open source wins." It's "the only layer that was soft just got harder, and everything underneath it is still hard."

Baker's own caveat is the honest part, and I'll quote it because it's the thing that keeps this from being hype:

> "This is not happening yet. Cheap, mostly open source tokens are likely the majority of volume today but the majority of economic value is still accruing to the most intelligent models. Might change though."[^baker]

Right. So the interesting question isn't *whether* the margin migrates. It's *how fast*, and *what it runs into on the way down*. Both of those turn out to be hardware questions.

### The catch-up is real, but not where the headlines put it

First, a correction I had to make to my own draft, because it changes the shape of the argument.

The tempting story is that K3 is cheap. It isn't, especially. On Artificial Analysis's own per-task numbers, K3 costs **$0.94 per benchmark task** versus GPT-5.6 Sol's **$1.04**.[^aa] That's parity, not disruption. K3 is genuinely more token-efficient than its predecessor (about 21% fewer output tokens than K2 across the eval suite), but it is not the thing undercutting the frontier on cost.

The disruption is a tier *below* K3:

| Model | Class | AA index | Cost per task |
|---|---|---:|---:|
| Claude Fable 5 | US closed | ~60 | premium |
| GPT-5.6 Sol | US closed | ~59 | $1.04 |
| Kimi K3 | open (CN) | ~57 | $0.94 |
| Grok 4.5 | US closed | ~54 | price challenger |
| GLM-5.2 | open (CN) | ~51 | $0.32 |
| DeepSeek V4 Pro | open (CN) | n/a | ~$0.04 |

GLM-5.2 at $0.32 and DeepSeek V4 Pro at roughly four cents are the real price war. They're not frontier-quality. They're "good enough for the routine 90% of what an enterprise actually routes," at a ~26x per-task gap to the frontier. That's the tier that tiered-routing shops deploy against the expensive models, and every routing rule written this year moves volume down-stack and doesn't move it back.

Now the number that keeps me honest in the other direction. Look at what "the majority of volume" actually cashes out to when you measure a paid production population instead of hobbyist traffic. Vercel published their AI Gateway index for July, and it's the cleanest instrument I've found:[^vercel]

| Measure | Open-weight | Closed + other |
|---|---:|---:|
| Token share, OpenRouter (all traffic) | ~61% | ~39% |
| Token share, Vercel production gateway | 29% | 71% |
| **Dollar share, Vercel production gateway** | **<4%** | **>96%** |

Read the last two rows together. Open models do 29% of the *work* in production and capture under 4% of the *money*. That's the volume/value paradox, measured. Both facts are true at once: erosion is real and fast (open-token share in production nearly tripled in two months), and value capture still overwhelmingly favors the frontier.

The gap between those two rows is the entire debate. If dollar share starts chasing token share, the base case is playing out. If token share stalls, frontier stickiness is winning. It's a monthly print now, not a vibe.

### Why this is a hardware story

Here's the part I actually care about, and the part the market seems least interested in pricing: open-source labs got good at this *because they were starved of memory*, and in engineering around that starvation they're rewriting what an inference machine needs to be.

Export controls kept Chinese labs short on high-bandwidth memory. So instead of buying their way out with more HBM, they attacked the thing that eats HBM: the KV cache. Three mechanisms, and they're not vaporware. The first two are in the papers and independently re-derivable.

**MLA: attention stops being a bandwidth problem.** DeepSeek's Multi-head Latent Attention compresses the key-value cache into a shared latent vector: about 68.6 KB per token, versus 4.5 MB for GPT-3-class multi-head attention.[^mla] That's a 67x cut, which means ~60x larger batches on the same silicon. With some layer reordering it also lifts the arithmetic intensity of decode by roughly 100x. Attention decode stops being memory-bound and starts being compute-bound. Remember that phrase.

**Kimi Delta Attention: the cache stops growing linearly.** K3 interleaves cheap linear-attention layers with full attention at a 3:1 ratio, so only a quarter of the layers keep a cache that grows with context.[^kda] Result: ~75% less KV memory, up to 6x decode throughput at a million tokens, and flat pricing across the entire context window as the commercial tell. Moonshot claims the hybrid actually *beats* full attention on quality rather than trading against it. That claim is self-reported and I'm holding it loosely, but the efficiency numbers are the load-bearing part and they hold.

Here's the whole ladder:

| Architecture | KV cache per token |
|---|---:|
| MHA (GPT-3 class) | 4.5 MB |
| GQA (modern baseline) | ~0.2–0.5 MB |
| MLA (DeepSeek) | 68.6 KB |
| KDA 3:1 hybrid (Kimi K3) | ~17 KB effective |

Stare at that column and the obvious trade is "short the memory makers." That's the third mechanism, and it's the one I want to be careful about, because I ran it down and it's wrong.

**The bottleneck migrates. It does not vanish.** The claim "MLA and MoE break the HBM-centric thesis" failed every version of the fact-check I threw at it. Here's why, and it's worth understanding as an engineer rather than taking on faith.

Whether inference is memory-bound or compute-bound comes down to **arithmetic intensity**: FLOPs performed per byte you drag out of memory. A GPU has two ceilings: how fast it can compute, and how fast it can read HBM. Their ratio is a break-even of a few hundred FLOPs per byte. Below that line you're memory-bound and the compute units sit idle waiting for data. Above it, memory keeps up and compute is the limit.

The lever on that ratio is **batch size**, and the reason is a beautiful asymmetry:

> The weights get read from HBM *once per forward pass*, no matter how many requests share it. The FLOPs scale with the number of requests. So arithmetic intensity rises roughly linearly with batch size.

At batch = 1 you drag a giant weight matrix out of memory to do one skinny matrix-vector multiply. Tiny FLOPs, huge bytes, hopelessly memory-bound. At batch = 256 you reuse each loaded weight across 256 requests and cross into compute-bound territory where HBM stops being the wall.

Now bring in sparse MoE, and here's the twist that kills the naive short. A dense model sends every token through every FFN weight, so a batch of 256 tokens all hit the same weights and batching is easy. A sparse MoE routes each token to a *few of 896* experts, so those 256 tokens get scattered across many experts and each expert sees a handful. The *effective* batch per expert collapses back toward 1 even when the nominal batch is large. You still have to read that expert's full weights from HBM, now amortized across almost nothing.

So extreme sparsity is FLOP-efficient and *sabotages batching*. The FFN goes memory-bound at batch sizes where a dense model would already be compute-bound. MLA fixes the KV-cache side of the ledger; it does nothing for the weight-reading side, and in a 2.8T-parameter model the weights are the elephant. The pressure on memory doesn't disappear. It shifts:

- **From bandwidth-per-FLOP to capacity-for-batching.** You need enough memory to hold huge weight pools *and* big batches so the experts stay full. "16 of 896 experts active" cuts FLOPs per token, but the full 2.8T weight pool still has to sit resident. Fewer active parameters is not a smaller machine.
- **From single-GPU memory to interconnect.** Spread experts across GPUs and routing generates irregular all-to-all traffic. The network becomes the constraint. That's the quiet bull case for switch and retimer silicon hiding inside the word "efficiency."
- **From raw bandwidth to compute density.** Once attention decode goes compute-bound, the ideal inference part carries more FLOPs per dollar and less exotic memory. That's exactly the custom-ASIC and SRAM-heavy design point.

One more correction while I'm being careful with numbers: the eye-catching "67x less memory" is MLA versus *ancient* GPT-3-era attention. Against the grouped-query attention everyone actually ships today, the KV advantage is a single-digit multiple, roughly 3–7x. Real, useful, not revolutionary. Another reason not to over-short memory on the architecture story.

So the honest read on memory is: hold it for the 2026–27 scarcity, which is bankable, and hedge the 2028 architecture risk, which is real. Two horizons. Don't confuse them. That's a nuance, not a short.

### Does cheaper intelligence mean less spend, or more?

Every conclusion downstream keys off one question. When the price of intelligence falls, does total spend rise or fall?

If demand is super-elastic, price wars at the model layer are a *gift* to everyone below it. If demand saturates, the whole capex complex is over-earning and the migration doesn't matter because the floor is falling out from under all of it.

The verified evidence leans hard toward elasticity. Goldman models tokens growing 24x by 2030, to something like 120 quadrillion a month, with inference cost-per-token deflating 60–70% a year, and hyperscaler gross margins *inflecting upward* because cost falls faster than price.[^goldman] Agentic workflows are the accelerant: a single agent interaction runs something like 30x the tokens of a chat turn, sometimes far more.[^ey]

But here's where I try not to fool myself: a chunk of today's token growth is waste and variance, not durable demand. On identical coding tasks, token consumption varies up to 30x run-to-run, accuracy peaks at *intermediate* spend and saturates past it, and models can't even predict their own usage.[^agent] Raw token counts are a noisy proxy. The thing to forecast is *successful tasks*, not tokens.

And the counterweight is arriving on schedule. Gartner expects over 40% of agentic AI projects to be canceled by end of 2027.[^gartner] Meta's Adam Mosseri said in mid-July that per-engineer token budgets are probably coming, right after Meta shut down an internal spend leaderboard that had pushed costs toward billions.[^mosseri] That's real. But read it precisely: it's rationalization of *waste*, a hype washout within adoption, not demand destruction. The same Gartner note still projects agentic AI in a third of enterprise software by 2028.

There's a trap in the elasticity story worth drawing out, because it's the crux of the whole complex:

| Price deflation | Volume growth needed to keep revenue flat |
|---|---:|
| 40% / yr | +67% |
| 50% / yr | +100% |
| 60% / yr | +150% |
| 70% / yr | +233% |

At Goldman's implied volume path, about +121% a year, token *revenue* only grows if realized *prices* deflate slower than ~55% a year. Goldman's 60–70% figure is *cost* deflation. The entire hyperscaler bull case lives in that wedge: cost falling 60–70% while price falls less. That gap *is* the gross-margin inflection.

Which means the metric that decides everything isn't token growth. It's whether open-source competition forces realized price to track cost 1:1. If it does, volume stops covering and the revenue line bends even as usage explodes. Watch realized dollars-per-million-tokens at the hyperscalers, not list prices.

### So where does the margin land?

I'm not going to hand you an order ticket. But the logic of the migration points somewhere specific, and it's worth saying plainly.

**The floors below the model win the base case.** Hyperscalers serve whichever model wins and inflect on margin as cost outruns price. The custom-ASIC duopoly (Broadcom and Marvell) is the purest expression: when the model is a commodity, serving cost is the entire game, and hyperscalers building their own silicon is how they win it. Broadcom's AI revenue was $10.8B last quarter, up 143%, guided to $16B next quarter.[^avgo] That's not a company waiting for the thesis. That's the thesis already running.

**NVIDIA is an inference company now, and the bear case misreads it.** "Open models kill training demand" misses that inference is already the majority of AI compute and the overwhelming majority of enterprise budget. Last quarter was $75.2B of data-center revenue, up 92%, and Jensen framed the whole thing around agentic inference.[^nvda] Cheap tokens are *volume*. The contested variable isn't NVIDIA's revenue, it's its gross-margin trajectory as custom ASICs take share. I'd own it as a margin-debate stock, not a monopoly.

**Neoclouds are a financing trade wearing an AI costume.** The question was never "will tokens grow." It's who carries the depreciation and the interest. A K3-style world is *bullish* their demand: the open-model serving stack runs anywhere, which erodes hyperscaler lock-in. But it also shortens how long any GPU vintage stays current, which shortens the depreciation schedule their debt was priced against. Own the balance sheet, rent the beta.

**Memory, as above: nuance, not a short.** **Power is the one floor that got worse every quarter.** Transformer lead times stretched from about two years to five, and it has pricing power in every scenario I can draw.

### Three ways it ends

I put rough probabilities on this. They're my judgment on the evidence, not anyone's model output. A second researcher working the same question independently landed within five points of these on the same tree, which I take as mild confirmation the tree is carved at the right joints, not that the numbers are right.

- **Gradual erosion (55%).** Open models stay 6–12 months behind and token-inefficient; the frontier keeps its quality premium but loses pricing power at the margin every quarter. The migration in slow motion. Everything below the model layer wins.
- **US pulls ahead (30%).** Recursive self-improvement compounds and the frontier labs convert compute into capability faster than open labs can distill it. Monopsony risk comes back; the labs squeeze their own suppliers again.
- **Outright loss (15%).** An open model matches the frontier *and* beats it on tokens-per-task. That's Baker's explicit "real Sputnik" trigger, the token-efficiency clause from the top of this piece. Closed-lab pricing collapses, lab training capex gets cut, inference demand keeps compounding on open weights.

Notice the book that falls out: overweight what wins in at least two of the three columns, the floors below the model, and avoid what needs one specific column to pay off.

### What I'm not sure about

In the spirit of not letting "estimated" quietly become "measured" in your head, here's the honest ledger.

I built this on a fan-out research pass: five search angles, ~22 sources, ~110 extracted claims, and the load-bearing ones run through three independent skeptic checks each. Some claims survived, some died, and I've tried to keep them visibly separate.

**Survived the fact-check:** Baker's posts and the exact quotes; K3's specs and pricing; the MLA and delta-attention efficiency numbers (independently re-derived from the papers); the Vercel volume/value split; Goldman's forecast; the NVIDIA and Broadcom prints; the agent-token variance research; the Meta spend-cap story.

**Died in verification. Do not reuse these:**

- "Frontier labs earn 90%+ inference margins." That's Baker's framing, not established fact.
- "MLA/MoE breaks the HBM thesis." The bottleneck migrates, as above.
- "Agentic AI is a ~1000x token multiplier." Enterprise-wide it's more like 10–100x, centered near 30x. The 1000x figure is real *only* for autonomous coding agents versus code chat, and it's driven by input tokens, which is why cache-hit pricing is the actual strategic battleground.
- "Chinese open models exert minimal pricing pressure." The complacent closed-lab bull case failed too. The pressure is real; it's the *pace* that's contested.

**Things I'm carrying on faith:** the neocloud drawdown levels, the exact ASIC-share numbers, and a McKinsey figure putting pure GPU-rental gross margins at 14–16% all come from single sources I couldn't fully verify. If that last one is right, a levered pure-rental neocloud is a bank with worse collateral, and it's the most important number in the whole neocloud leg. I flag it because I want it to be true and that's exactly when to distrust yourself.

And the biggest known unknown: my "Wall Street proxy" is essentially one investor. Brad Gerstner, the other voice I went looking for, posted nothing on K3 in this window that survived verification. One well-sourced opinion is not a consensus.

That's the note. The one line I'd keep if I had to throw the rest away:

> When the scarce thing stops being scarce, don't ask who loses. Ask which floor the margin falls to, and whether that floor is still hard.

*Not investment advice. I hold positions in some of the names discussed, my confidence is uneven and labeled, and I'd rather be usefully wrong in public than vaguely right in private. If you think I've got the mechanism wrong, especially the batching argument, tell me. That's more useful than agreement.*

[^k3]: Moonshot AI, ["Kimi K3"](https://www.kimi.com/blog/kimi-k3) (blog, July 16 2026); model card on [OpenRouter](https://openrouter.ai/moonshotai/kimi-k3). Performance and efficiency figures are vendor-reported pending the July 27 open-weight release and independent replication at 2.8T scale.
[^aa]: Artificial Analysis, ["Kimi K3 in the Artificial Analysis Intelligence Index"](https://artificialanalysis.ai/articles/kimi-k3-achieves-3-in-the-artificial-analysis-intelligence-index-comparable-to-opus-4-8-and-gpt-5-5/): index scores and per-task cost, including K3 at $0.94/task vs GPT-5.6 Sol at $1.04.
[^baker]: Gavin Baker on X, [July 17 2026](https://x.com/GavinSBaker/status/2078110934740980193) and [July 12](https://x.com/GavinSBaker/status/2076369936251851091). X blocked direct fetch; quotes triangulated from index snippets and two independent write-ups.
[^vercel]: Vercel, ["AI Gateway Production Index, July 2026"](https://vercel.com/blog/ai-gateway-production-index-july-2026). Open-weight at 29% of production tokens and under 4% of spend; DeepSeek alone 22.6% of token volume. OpenRouter's ~61% figure is a hobbyist/arbitrage-heavy population and is directionally, not precisely, sourced.
[^mla]: Arithmetic-intensity and KV-cache figures from [arXiv 2507.15465](https://arxiv.org/html/2507.15465) and [2506.02523](https://arxiv.org/abs/2506.02523); re-derived rather than taken on the authors' word. The GQA-vs-MLA 3–7x range is verified against the same work.
[^kda]: Kimi Linear / Kimi Delta Attention: [arXiv 2510.26692](https://arxiv.org/pdf/2510.26692). The −75% KV and 6x decode numbers are Moonshot's, at a matched 48B scale; the "beats full attention" Pareto claim is the weakest link and I treat it as such.
[^goldman]: Goldman Sachs Research, ["AI agents forecast to boost tech cash flow as usage soars"](https://www.goldmansachs.com/insights/articles/ai-agents-forecast-to-boost-tech-cash-flow-as-usage-soars) (May 20 2026).
[^ey]: EY, ["Agentic AI token costs"](https://www.ey.com/en_us/insights/ai/agentic-ai-token-costs). The ~30x figure is a center estimate; production ranges run 10–100x.
[^agent]: ["Forecasting agent token usage"](https://arxiv.org/abs/2604.22750): 30x run-to-run variance on identical tasks, accuracy peaking at intermediate spend, and self-prediction correlation ≤0.39. Also the source for the narrow, input-driven ~1000x autonomous-coding figure.
[^gartner]: Gartner, ["Over 40% of agentic AI projects will be canceled by end of 2027"](https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027).
[^mosseri]: TechCrunch, ["Meta's Adam Mosseri says AI token budgets could soon be capped per engineer"](https://techcrunch.com/2026/07/14/metas-adam-mosseri-says-ai-token-budgets-could-soon-be-capped-per-engineer/) (July 14 2026).
[^nvda]: [NVIDIA Q1 FY2027 results](https://investor.nvidia.com/news/press-release-details/2026/NVIDIA-Announces-Financial-Results-for-First-Quarter-Fiscal-2027/default.aspx): $75.2B data-center revenue (+92% YoY), $81.6B total, framed around agentic inference and the Dynamo 1.0 serving stack.
[^avgo]: [Broadcom Q2 FY2026 results](https://www.prnewswire.com/news-releases/broadcom-inc-announces-second-quarter-fiscal-year-2026-financial-results-and-quarterly-dividend-302790698.html): $10.8B AI revenue (+143% YoY), Q3 guide of $16B, FY26 target $56B.
