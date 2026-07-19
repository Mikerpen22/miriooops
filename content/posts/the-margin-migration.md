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

AI economics makes more sense to me when I look for the bottleneck. Today it might be model quality; tomorrow it might be memory bandwidth, power, or a transformer that takes five years to deliver. Whoever controls the scarce part gets to keep more of the money.

Kimi K3 made me rethink where that bottleneck is moving.

### Seventy-two hours in July

On July 16, Moonshot AI shipped **Kimi K3**: a 2.8-trillion-parameter sparse mixture-of-experts model that activates 16 of its 896 experts per token, supports a million-token context, and will be released with open weights.[^k3] It scores about 57 on Artificial Analysis's intelligence index. Claude Fable 5 scores about 60 and GPT-5.6 Sol about 59.[^aa] An open-weight model from a Chinese lab is now within roughly three points of the leading closed models.

The next day, tech investor Gavin Baker described the possible market impact:

> "Kimi K3 may be an important inflection point for AI. Potentially negative for Anthropic and OpenAI while being net positive for essentially every other company in the world... the real Sputnik moment would be an open-source frontier model that was also token efficient."
>
> — Gavin Baker, X, July 17 2026[^baker]

The last two words matter. Open weights are not automatically cheap weights, especially when the model has 2.8 trillion parameters.

K3 raises two separate questions. How close can an open model get to the frontier, and what does it cost to run? It narrows the first gap much more than the second. But each time the model itself becomes easier to replace, bargaining power shifts toward the infrastructure that runs it. That is why I see K3 as an infrastructure release too.

### When the model layer loses pricing power

Decompose the price of an AI token and you get, roughly: **power + silicon + data center + capital + model margin.**

For the last two years, a few closed labs have controlled the best models. Their scale makes them important buyers of chips, power, and data-center capacity. The model layer has kept software-like margins because cheaper models could not match its quality.

Near-frontier open weights put pressure on that pricing power. More providers can serve similar models, so the model itself becomes easier to substitute. Meanwhile, every deployment still needs cloud capacity, silicon, memory, power, and interconnect. Those inputs remain scarce regardless of whose weights are loaded.

If cheaper tokens create enough new usage, infrastructure providers can capture part of the value that model vendors lose. The catch is that usage has to grow faster than prices fall.

That is the margin migration: less pricing power in the model, with the remaining scarcity lower in the stack.

Baker was careful not to claim that this shift had already happened:

> "This is not happening yet. Cheap, mostly open source tokens are likely the majority of volume today but the majority of economic value is still accruing to the most intelligent models. Might change though."[^baker]

Whether that shift actually happens comes down to inference hardware, serving costs, and demand.

### K3 narrows the quality gap, not the cost gap

My first read was that K3 had closed both the quality gap and the cost gap. It has not. On Artificial Analysis's per-task numbers, K3 costs **$0.94 per benchmark task**, compared with **$1.04** for GPT-5.6 Sol.[^aa] That is close to price parity, not a price break. K3 uses about 21% fewer output tokens than K2 across the evaluation suite, but it is not undercutting the frontier on cost.

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

Vercel's July AI Gateway index makes the remaining gap unusually clear because it separates token volume from spending:[^vercel]

| Measure | Open-weight | Closed + other |
|---|---:|---:|
| Token share, OpenRouter (all traffic) | ~61% | ~39% |
| Token share, Vercel production gateway | 29% | 71% |
| **Dollar share, Vercel production gateway** | **<4%** | **>96%** |

Open models account for 29% of production tokens but less than 4% of spending. Their production token share nearly tripled in two months, while the overwhelming majority of revenue still went to closed models.

That gap between token share and dollar share is the number I would watch. If the two begin to converge, closed-model pricing power is weakening. If they do not, customers are still happy to pay a large premium for frontier quality.

### Why this is a hardware story

This is where K3 gets interesting to me. Export controls limited Chinese labs' access to high-bandwidth memory. Instead of waiting for more HBM, they attacked one of its largest consumers: the key-value (KV) cache.

**MLA reduces KV-cache bandwidth.** DeepSeek's Multi-head Latent Attention compresses the key-value cache into a shared latent vector: about 68.6 KB per token, versus 4.5 MB for GPT-3-class multi-head attention.[^mla] That is a 67x reduction and allows batches roughly 60 times larger on the same silicon. With some layer reordering, it can also raise the arithmetic intensity of decoding by about 100x, moving attention decode from a memory bottleneck toward a compute bottleneck.

**Kimi Delta Attention slows cache growth.** K3 interleaves linear-attention layers with full attention at a 3:1 ratio, so only a quarter of the layers keep a cache that grows with context.[^kda] Moonshot reports about 75% less KV memory and up to six times the decode throughput at a million tokens. The company also says the hybrid matches or beats full attention on quality, although that claim still needs independent testing at K3's scale.

Put side by side, the difference is large:

| Architecture | KV cache per token |
|---|---:|
| MHA (GPT-3 class) | 4.5 MB |
| GQA (modern baseline) | ~0.2–0.5 MB |
| MLA (DeepSeek) | 68.6 KB |
| KDA 3:1 hybrid (Kimi K3) | ~17 KB effective |

This sounds like bad news for memory suppliers until you follow the bottleneck to its next stop.

**The cache shrinks, but the weights remain.** MLA reduces KV-cache pressure, while sparse mixture-of-experts (MoE) models still need to read very large sets of model weights. Arithmetic intensity explains why this matters.

Whether inference is memory-bound or compute-bound comes down to **arithmetic intensity**: how much math the GPU can do for every byte it pulls from memory. A GPU has two ceilings, one for computation and one for HBM bandwidth. Below their break-even point, the compute units sit idle waiting for data. Above it, memory keeps up and computation becomes the limit.

The main lever on that ratio is **batch size**. The weights are read from HBM once per forward pass, no matter how many requests share it, while the FLOPs scale with the number of requests. Arithmetic intensity therefore rises roughly in line with batch size.

At a batch size of 1, the system loads a large weight matrix to perform one matrix-vector multiplication. There is little computation relative to the amount of data moved, so inference is memory-bound. At a batch size of 256, the system can reuse those weights across 256 requests and may become compute-bound instead.

Sparse MoE makes that reuse harder. A dense model sends every token through the same feed-forward-network weights, so all 256 tokens in a batch reuse them. A sparse MoE routes each token to a few of 896 experts. The tokens are scattered, each expert may see only a small batch, and its weights still have to be read from HBM. The nominal batch may be large while the effective batch per expert remains small.

Extreme sparsity saves computation but reduces the benefit of batching. The feed-forward network can remain memory-bound at batch sizes where a dense model would already be compute-bound. MLA reduces the KV cache; it does not reduce the cost of reading the weights of a 2.8-trillion-parameter model. Memory pressure changes form:

- **From bandwidth-per-FLOP to capacity-for-batching.** You need enough memory to hold huge weight pools *and* big batches so the experts stay full. "16 of 896 experts active" cuts FLOPs per token, but the full 2.8T weight pool still has to sit resident. Fewer active parameters is not a smaller machine.
- **From single-GPU memory to interconnect.** When experts are spread across GPUs, routing creates irregular all-to-all traffic. The network can become the constraint, increasing the importance of switch and retimer silicon.
- **From raw bandwidth to compute density.** Once attention decode goes compute-bound, the ideal inference part carries more FLOPs per dollar and less exotic memory. That's exactly the custom-ASIC and SRAM-heavy design point.

The "67x less memory" headline also flatters the comparison because the baseline is GPT-3-era attention. Against the grouped-query attention used in current models, MLA's KV-cache advantage is closer to 3–7x. Still useful, just less miraculous.

For memory vendors, the timing is awkward: supply is tight in 2026–27, but the architectures being developed during that shortage may reduce how much memory future systems need.

### Does cheaper intelligence mean less spend, or more?

The rest of the argument depends on one question: when the price of intelligence falls, does total spending rise or fall?

If usage grows faster than prices fall, infrastructure providers benefit. If demand saturates, the industry has built too much capacity and there may be little margin to redistribute.

Goldman models token usage growing 24-fold by 2030, to about 120 quadrillion tokens a month, while inference cost per token falls 60–70% a year. In that model, hyperscaler gross margins rise because costs fall faster than prices.[^goldman] Agent workflows add another source of demand: one agent interaction may use about 30 times as many tokens as a chat turn, and sometimes much more.[^ey]

Some of today's token growth is waste rather than durable demand. On identical coding tasks, token consumption can vary by as much as 30 times from run to run. Accuracy peaks at an intermediate level of spending and then saturates, and models are poor at predicting their own usage.[^agent] Successful tasks are therefore a better demand measure than raw token counts.

The cleanup has already started. Gartner expects more than 40% of agentic AI projects to be canceled by the end of 2027.[^gartner] Meta's Adam Mosseri said in mid-July that per-engineer token budgets are likely, after Meta shut down an internal spending leaderboard that had pushed projected costs toward billions.[^mosseri] That looks more like the end of unlimited experimentation than the end of demand. The same Gartner note projects agentic AI in a third of enterprise software by 2028.

But 24-fold token growth does not guarantee revenue growth. The price decline still has to be beaten:

| Price deflation | Volume growth needed to keep revenue flat |
|---|---:|
| 40% / yr | +67% |
| 50% / yr | +100% |
| 60% / yr | +150% |
| 70% / yr | +233% |

Goldman's implied volume growth is about 121% a year. At that rate, token *revenue* grows only if realized *prices* fall by less than about 55% a year. Goldman's 60–70% figure refers to *cost* reduction. Hyperscaler margins improve only if costs continue to fall faster than prices.

What matters is whether open-model competition forces realized prices down as quickly as serving costs fall. If it does, usage can soar while revenue growth slows. Realized revenue per million tokens tells us more than advertised list prices.

### So where does the margin land?

**Infrastructure providers benefit if demand remains elastic.** Hyperscalers can serve whichever model customers choose, and they gain margin if serving costs fall faster than prices. Custom ASICs from Broadcom and Marvell matter because hyperscalers use their own silicon to reduce serving costs. Broadcom reported $10.8 billion in quarterly AI revenue, up 143%, and guided to $16 billion for the next quarter.[^avgo]

**NVIDIA increasingly depends on inference, not only training.** The claim that open models will reduce training demand misses the growing amount of compute used for inference. NVIDIA reported $75.2 billion in quarterly data-center revenue, up 92%, and Jensen Huang emphasized agentic inference.[^nvda] Cheaper tokens can increase inference volume. The harder question is what happens to NVIDIA's gross margin as custom ASICs take share.

**For neoclouds, financing matters as much as demand.** Open models can run outside the large hyperscalers, which creates demand for independent GPU providers. But new hardware also makes older GPU fleets less competitive. The important variables are therefore debt, interest expense, and whether the useful life of the hardware matches its depreciation schedule.

**Memory vendors face the short- and long-term tension described above.** **Power equipment has a simpler setup:** transformer lead times have reportedly stretched from about two years to five, which gives suppliers pricing power across most demand scenarios.

### Three scenarios

I can see three ways this develops.

- **Gradual erosion seems most likely.** Open models remain 6–12 months behind and less token-efficient. Frontier labs keep a quality premium, but the premium shrinks as more work moves to cheaper models. Infrastructure demand continues to grow.
- **The US labs pull away again.** Frontier labs turn compute into capability faster than open labs can copy or distill their results. Their models remain hard to substitute, preserving both pricing power and leverage over suppliers.
- **An open model reaches the frontier and runs more efficiently.** Closed-model pricing falls quickly, training budgets come under pressure, and more inference moves to open weights.

In all three cases, the lower layers depend less on which model vendor wins. That is the appeal.

### Where this could be wrong

I may be overestimating how much model margin exists in the first place. Chips, power, networking, reliability, sales, and safety all sit behind the price of a token. If those costs already consume most of the revenue, lower model prices hurt the whole stack rather than enriching the layers below it.

I may also be underestimating the engineers. MLA, MoE, and attention optimizations relieve one bottleneck and expose others, especially batching, interconnect, and utilization. If they work through those constraints faster than agent workloads expand, the industry will need less hardware than today's forecasts assume.

And I may be early. A cheaper model can appear overnight; enterprise spending does not move that quickly. Reliability, procurement, compliance, and existing product integrations can protect an incumbent long after its technical lead narrows.

K3 has not collapsed model pricing. What changed for me is simpler: I no longer assume frontier quality will stay scarce just because it is scarce now. Models can be copied and improved quickly. Power, memory, interconnect, data-center capacity, and distribution move much more slowly.

*Not investment advice. I hold positions in some of the companies discussed.*

[^k3]: Moonshot AI, ["Kimi K3"](https://www.kimi.com/blog/kimi-k3) (blog, July 16 2026); model card on [OpenRouter](https://openrouter.ai/moonshotai/kimi-k3). Performance and efficiency figures are vendor-reported pending the July 27 open-weight release and independent replication at 2.8T scale.
[^aa]: Artificial Analysis, ["Kimi K3 in the Artificial Analysis Intelligence Index"](https://artificialanalysis.ai/articles/kimi-k3-achieves-3-in-the-artificial-analysis-intelligence-index-comparable-to-opus-4-8-and-gpt-5-5/): index scores and per-task cost, including K3 at $0.94/task vs GPT-5.6 Sol at $1.04.
[^baker]: Gavin Baker on X, [July 17 2026](https://x.com/GavinSBaker/status/2078110934740980193) and [July 12](https://x.com/GavinSBaker/status/2076369936251851091).
[^vercel]: Vercel, ["AI Gateway Production Index, July 2026"](https://vercel.com/blog/ai-gateway-production-index-july-2026). Open-weight models accounted for 29% of production tokens and less than 4% of spending; DeepSeek alone accounted for 22.6% of token volume.
[^mla]: Arithmetic-intensity and KV-cache figures from [arXiv 2507.15465](https://arxiv.org/html/2507.15465) and [arXiv 2506.02523](https://arxiv.org/abs/2506.02523).
[^kda]: Kimi Linear / Kimi Delta Attention: [arXiv 2510.26692](https://arxiv.org/pdf/2510.26692). The 75% KV reduction and 6x decode figures are Moonshot's measurements at a matched 48B scale.
[^goldman]: Goldman Sachs Research, ["AI agents forecast to boost tech cash flow as usage soars"](https://www.goldmansachs.com/insights/articles/ai-agents-forecast-to-boost-tech-cash-flow-as-usage-soars) (May 20 2026).
[^ey]: EY, ["Agentic AI token costs"](https://www.ey.com/en_us/insights/ai/agentic-ai-token-costs). The ~30x figure is a center estimate; production ranges run 10–100x.
[^agent]: ["Forecasting agent token usage"](https://arxiv.org/abs/2604.22750): 30x run-to-run variance on identical tasks, accuracy peaking at intermediate spend, and self-prediction correlation ≤0.39. Also the source for the narrow, input-driven ~1000x autonomous-coding figure.
[^gartner]: Gartner, ["Over 40% of agentic AI projects will be canceled by end of 2027"](https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027).
[^mosseri]: TechCrunch, ["Meta's Adam Mosseri says AI token budgets could soon be capped per engineer"](https://techcrunch.com/2026/07/14/metas-adam-mosseri-says-ai-token-budgets-could-soon-be-capped-per-engineer/) (July 14 2026).
[^nvda]: [NVIDIA Q1 FY2027 results](https://investor.nvidia.com/news/press-release-details/2026/NVIDIA-Announces-Financial-Results-for-First-Quarter-Fiscal-2027/default.aspx): $75.2B data-center revenue (+92% YoY), $81.6B total, framed around agentic inference and the Dynamo 1.0 serving stack.
[^avgo]: [Broadcom Q2 FY2026 results](https://www.prnewswire.com/news-releases/broadcom-inc-announces-second-quarter-fiscal-year-2026-financial-results-and-quarterly-dividend-302790698.html): $10.8B AI revenue (+143% YoY), Q3 guide of $16B, FY26 target $56B.
