---
author:
  name: "Minhung Shih"
date: 2026-07-18
linktitle: "The Margin Migration"
title: "The Margin Migration: what Kimi K3 actually repriced"
description: "Kimi K3 came within three points of the closed-model frontier, while cheaper open models widened the price gap below it. What happens to AI margins when the model itself becomes easier to substitute?"
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

I spend most of my day thinking about systems: where the bottleneck is, who pays for it, and what changes when the bottleneck moves. The same questions apply to the companies that turn electricity into AI tokens.

Kimi K3 changed my view of that stack. I started writing this post to work out what, exactly, had changed.

This is not investment advice. Some of the evidence is solid; some comes from a single source and should be treated accordingly. I list the important uncertainties at the end.

### Seventy-two hours in July

On July 16, Moonshot AI shipped **Kimi K3**: a 2.8-trillion-parameter sparse mixture-of-experts model that activates 16 of its 896 experts per token, supports a million-token context, and will be released with open weights.[^k3] It scores about 57 on Artificial Analysis's intelligence index. Claude Fable 5 scores about 60 and GPT-5.6 Sol about 59.[^aa] An open-weight model from a Chinese lab is now within roughly three points of the leading closed models.

The next day, tech investor Gavin Baker described the possible market impact:

> "Kimi K3 may be an important inflection point for AI. Potentially negative for Anthropic and OpenAI while being net positive for essentially every other company in the world... the real Sputnik moment would be an open-source frontier model that was also token efficient."
>
> — Gavin Baker, X, July 17 2026[^baker]

The important qualification is "token efficient." Open weights alone do not make a model cheap to run.

The usual interpretation is "open source wins, closed labs lose." I think that misses the more interesting change:

> As models become easier to substitute, some of the margin moves into the infrastructure that runs them.

That is why I see K3 as an infrastructure story, not just a model-release story.

### When the model layer loses pricing power

Decompose the price of an AI token and you get, roughly: **power + silicon + data center + capital + model margin.**

For the last two years, a few closed labs have controlled the best models. Their scale makes them important buyers of chips, power, and data-center capacity, although they do not control every supply-constrained market below them. The model layer has kept software-like margins because cheaper models could not match its quality.

Near-frontier open weights put pressure on that pricing power. More providers can serve similar models, so the model itself becomes easier to substitute. Meanwhile, every deployment still needs cloud capacity, silicon, memory, power, and interconnect. Those inputs remain scarce regardless of whose weights are loaded.

If cheaper tokens create enough new usage, infrastructure providers can capture part of the value that the model vendors lose. This depends on demand growing faster than prices fall; it is not automatic.

That is the margin migration: less pricing power in the model, with the remaining scarcity lower in the stack.

Baker also included an important caveat:

> "This is not happening yet. Cheap, mostly open source tokens are likely the majority of volume today but the majority of economic value is still accruing to the most intelligent models. Might change though."[^baker]

The open question is how quickly value moves, if it moves at all. The answer depends heavily on inference hardware and serving costs.

### K3 narrows the quality gap, not the cost gap

I had to correct an earlier version of this argument.

K3 is not especially cheap. On Artificial Analysis's per-task numbers, it costs **$0.94 per benchmark task**, compared with **$1.04** for GPT-5.6 Sol.[^aa] That is close to price parity, not a major price break. K3 uses about 21% fewer output tokens than K2 across the evaluation suite, but it is not undercutting the frontier on cost.

The larger cost gap appears in the tier *below* K3:

| Model | Class | AA index | Cost per task |
|---|---|---:|---:|
| Claude Fable 5 | US closed | ~60 | premium |
| GPT-5.6 Sol | US closed | ~59 | $1.04 |
| Kimi K3 | open (CN) | ~57 | $0.94 |
| Grok 4.5 | US closed | ~54 | price challenger |
| GLM-5.2 | open (CN) | ~51 | $0.32 |
| DeepSeek V4 Pro | open (CN) | n/a | ~$0.04 |

GLM-5.2 at $0.32 and DeepSeek V4 Pro at roughly four cents show where the price competition is happening. They are not frontier models, but they may be good enough for routine work that does not need the best available model. DeepSeek's per-task cost is about 4% of GPT-5.6 Sol's, which gives companies a strong reason to route simpler requests away from premium models.

The production data also shows how far open models still have to go. Vercel's July AI Gateway index separates token volume from actual spending:[^vercel]

| Measure | Open-weight | Closed + other |
|---|---:|---:|
| Token share, OpenRouter (all traffic) | ~61% | ~39% |
| Token share, Vercel production gateway | 29% | 71% |
| **Dollar share, Vercel production gateway** | **<4%** | **>96%** |

Open models account for 29% of production tokens but less than 4% of spending. Their production token share nearly tripled in two months, while the overwhelming majority of revenue still went to closed models.

The gap between token share and dollar share is the number to watch. If open models begin taking a comparable share of spending, the migration is under way. If their token share stalls, customers are still willing to pay a large premium for frontier quality.

### Why this is a hardware story

The hardware changes are the part I find most interesting. Limited access to high-bandwidth memory pushed Chinese labs to reduce how much memory inference requires. Their solutions are changing the design of inference systems.

Export controls limited Chinese labs' access to high-bandwidth memory. Instead of relying on more HBM, they reduced one of its largest consumers: the key-value (KV) cache. Two published techniques are especially relevant.

**MLA reduces KV-cache bandwidth.** DeepSeek's Multi-head Latent Attention compresses the key-value cache into a shared latent vector: about 68.6 KB per token, versus 4.5 MB for GPT-3-class multi-head attention.[^mla] That is a 67x reduction and allows batches roughly 60 times larger on the same silicon. With some layer reordering, it can also raise the arithmetic intensity of decoding by about 100x, moving attention decode from a memory bottleneck toward a compute bottleneck.

**Kimi Delta Attention slows cache growth.** K3 interleaves linear-attention layers with full attention at a 3:1 ratio, so only a quarter of the layers keep a cache that grows with context.[^kda] Moonshot reports about 75% less KV memory and up to six times the decode throughput at a million tokens. It also claims that the hybrid matches or beats full attention on quality. That quality result is self-reported, so I give it less weight than the memory and throughput measurements.

The reductions look like this:

| Architecture | KV cache per token |
|---|---:|
| MHA (GPT-3 class) | 4.5 MB |
| GQA (modern baseline) | ~0.2–0.5 MB |
| MLA (DeepSeek) | 68.6 KB |
| KDA 3:1 hybrid (Kimi K3) | ~17 KB effective |

At first glance, this looks like bad news for memory suppliers. That conclusion ignores where the next bottleneck appears.

**The bottleneck moves; it does not disappear.** MLA reduces KV-cache pressure, but sparse mixture-of-experts (MoE) models still need to read very large sets of model weights. Arithmetic intensity explains why this matters.

Whether inference is memory-bound or compute-bound comes down to **arithmetic intensity**: FLOPs performed per byte you drag out of memory. A GPU has two ceilings: how fast it can compute, and how fast it can read HBM. Their ratio is a break-even of a few hundred FLOPs per byte. Below that line you're memory-bound and the compute units sit idle waiting for data. Above it, memory keeps up and compute is the limit.

The main lever on that ratio is **batch size**:

> The weights get read from HBM *once per forward pass*, no matter how many requests share it. The FLOPs scale with the number of requests. So arithmetic intensity rises roughly linearly with batch size.

At a batch size of 1, the system loads a large weight matrix to perform one matrix-vector multiplication. There is little computation relative to the amount of data moved, so inference is memory-bound. At a batch size of 256, the system can reuse those weights across 256 requests and may become compute-bound instead.

Sparse MoE makes that reuse harder. A dense model sends every token through the same feed-forward-network weights, so all 256 tokens in a batch reuse them. A sparse MoE routes each token to a few of 896 experts. The tokens are scattered, each expert may see only a small batch, and its weights still have to be read from HBM. The nominal batch may be large while the effective batch per expert remains small.

Extreme sparsity saves computation but reduces the benefit of batching. The feed-forward network can remain memory-bound at batch sizes where a dense model would already be compute-bound. MLA reduces the KV cache; it does not reduce the cost of reading the weights of a 2.8-trillion-parameter model. Memory pressure changes form:

- **From bandwidth-per-FLOP to capacity-for-batching.** You need enough memory to hold huge weight pools *and* big batches so the experts stay full. "16 of 896 experts active" cuts FLOPs per token, but the full 2.8T weight pool still has to sit resident. Fewer active parameters is not a smaller machine.
- **From single-GPU memory to interconnect.** When experts are spread across GPUs, routing creates irregular all-to-all traffic. The network can become the constraint, increasing the importance of switch and retimer silicon.
- **From raw bandwidth to compute density.** Once attention decode goes compute-bound, the ideal inference part carries more FLOPs per dollar and less exotic memory. That's exactly the custom-ASIC and SRAM-heavy design point.

The headline "67x less memory" also needs context: it compares MLA with GPT-3-era attention. Against the grouped-query attention common in current models, the KV-cache advantage is roughly 3–7x. That is useful, but much less dramatic.

My conclusion on memory has two time horizons. Current supply constraints support demand in 2026–27, while architectural changes could reduce memory requirements later. The near-term shortage and the longer-term design risk can both be true.

### Does cheaper intelligence mean less spend, or more?

The rest of the argument depends on one question: when the price of intelligence falls, does total spending rise or fall?

If usage grows faster than prices fall, infrastructure providers benefit. If demand saturates, the industry has built too much capacity and there may be little margin to redistribute.

The evidence I found points toward strong demand growth. Goldman models token usage growing 24-fold by 2030, to about 120 quadrillion tokens a month, while inference cost per token falls 60–70% a year. Its model has hyperscaler gross margins rising because costs fall faster than prices.[^goldman] Agent workflows add to that demand: one agent interaction may use about 30 times as many tokens as a chat turn, and sometimes much more.[^ey]

Some of today's token growth is waste rather than durable demand. On identical coding tasks, token consumption can vary by as much as 30 times from run to run. Accuracy peaks at an intermediate level of spending and then saturates, and models are poor at predicting their own usage.[^agent] Successful tasks are therefore a better demand measure than raw token counts.

Companies are also starting to control waste. Gartner expects more than 40% of agentic AI projects to be canceled by the end of 2027.[^gartner] Meta's Adam Mosseri said in mid-July that per-engineer token budgets are likely, after Meta shut down an internal spending leaderboard that had pushed projected costs toward billions.[^mosseri] I read this as tighter cost control, not necessarily falling demand. The same Gartner note projects agentic AI in a third of enterprise software by 2028.

The arithmetic still sets a demanding threshold:

| Price deflation | Volume growth needed to keep revenue flat |
|---|---:|
| 40% / yr | +67% |
| 50% / yr | +100% |
| 60% / yr | +150% |
| 70% / yr | +233% |

Goldman's implied volume growth is about 121% a year. At that rate, token *revenue* grows only if realized *prices* fall by less than about 55% a year. Goldman's 60–70% figure refers to *cost* reduction. Hyperscaler margins improve only if costs continue to fall faster than prices.

Token growth alone is not enough. The key metric is whether open-model competition forces realized prices to fall at the same rate as costs. If it does, usage can grow rapidly while revenue growth slows. Realized revenue per million tokens is more informative than advertised list prices.

### So where does the margin land?

The argument has different implications for each part of the infrastructure stack.

**Infrastructure providers benefit if demand remains elastic.** Hyperscalers can serve whichever model customers choose, and they gain margin if serving costs fall faster than prices. Custom ASICs from Broadcom and Marvell matter because hyperscalers use their own silicon to reduce serving costs. Broadcom reported $10.8 billion in quarterly AI revenue, up 143%, and guided to $16 billion for the next quarter.[^avgo]

**NVIDIA increasingly depends on inference, not only training.** The claim that open models will reduce training demand misses the growing amount of compute used for inference. NVIDIA reported $75.2 billion in quarterly data-center revenue, up 92%, and Jensen Huang emphasized agentic inference.[^nvda] Cheaper tokens can increase inference volume. The harder question is what happens to NVIDIA's gross margin as custom ASICs take share.

**For neoclouds, financing matters as much as demand.** Open models can run outside the large hyperscalers, which creates demand for independent GPU providers. But new hardware also makes older GPU fleets less competitive. The important variables are therefore debt, interest expense, and whether the useful life of the hardware matches its depreciation schedule.

**Memory has different short- and long-term risks, as discussed above.** **Power equipment remains constrained.** Transformer lead times have reportedly stretched from about two years to five, giving suppliers pricing power across most demand scenarios.

### Three scenarios

These probabilities are my own estimates, not model output or market forecasts. Another researcher reviewing the same evidence produced estimates within five percentage points of mine. That made me more comfortable with the three scenarios, but not with the exact probabilities.

- **Gradual erosion (55%).** Open models remain 6–12 months behind and less token-efficient. Frontier labs keep a quality premium but gradually lose pricing power. Infrastructure demand continues to grow.
- **US pulls ahead (30%).** Frontier labs turn compute into capability faster than open labs can copy or distill their results. Their models remain difficult to substitute, preserving their pricing power and leverage over suppliers.
- **Outright loss (15%).** An open model matches the frontier and uses fewer tokens per task. This is Baker's "real Sputnik" condition. Closed-lab pricing falls sharply, training capital expenditure is cut, and inference demand continues to grow on open weights.

Across these scenarios, infrastructure is less dependent on any one model vendor winning. That is the main reason I prefer the lower layers of the stack.

### What I'm not sure about

Several parts of this argument are better supported than others.

I reviewed about 22 sources and checked the claims that materially affect the conclusion against multiple sources where possible.

**Claims I could verify:** Baker's posts and the quoted text; K3's specifications and pricing; the MLA and delta-attention efficiency numbers; Vercel's token-volume and spending split; Goldman's forecast; NVIDIA's and Broadcom's reported results; the research on agent token variance; and Meta's discussion of spending caps.

**Claims I rejected after checking:**

- "Frontier labs earn 90%+ inference margins." That's Baker's framing, not established fact.
- "MLA/MoE breaks the HBM thesis." It reduces one source of memory pressure but creates or exposes others.
- "Agentic AI is a ~1000x token multiplier." Enterprise-wide it's more like 10–100x, centered near 30x. The 1000x figure is real *only* for autonomous coding agents versus code chat, and it's driven by input tokens, which is why cache-hit pricing is the actual strategic battleground.
- "Chinese open models exert minimal pricing pressure." The pressure is visible; the uncertain part is how quickly it will affect revenue.

**Claims I could not verify independently:** the neocloud drawdown levels, the exact ASIC market-share figures, and a McKinsey estimate that puts gross margins for pure GPU rental at 14–16%. I have not used those figures as primary support for the conclusion. The McKinsey estimate would materially weaken the case for highly leveraged GPU-rental businesses, so it needs better evidence before I rely on it.

I also found only one investor, Gavin Baker, making the specific market argument quoted in this post. I looked for a comparable view from Brad Gerstner but could not verify one from this period. Baker's view is useful, but it is not evidence of a Wall Street consensus.

The shortest version of my argument is:

> When scarcity moves to another part of the stack, margins tend to move with it.

*Not investment advice. I hold positions in some of the companies discussed. If the batching argument or another technical mechanism is wrong, I would like to know; the thesis depends more on those mechanisms than on the market labels.*

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
