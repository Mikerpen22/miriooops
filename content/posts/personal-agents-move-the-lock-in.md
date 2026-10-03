---
author:
  name: "Minhung Shih"
date: 2026-10-03
draft: false
linktitle: "Personal Agents Move the Lock-In"
title: "Personal Agents Move the Lock-In"
description: "Personal agents could make services easier to replace while becoming harder to replace themselves. Amazon, Shopify, and Apple show why the incentives differ."
type:
- post
- posts
series:
- Markets
tags:
- agents
- ai
- platforms
- markets
---

Suppose I tell a personal agent to buy groceries for tomorrow, stay under $80, and remember that one of my friends cannot eat peanuts.

Today, I would probably open the store I already use. I know its search, I have an account, and I know what happens if something arrives damaged. Comparing another store costs time even when it might save money.

A capable agent could do that comparison for me. It could check several stores, choose substitutions, and ask me to approve the order. The store's interface would matter less. Its prices, inventory, delivery, and reliability would matter more.

That is what interests me about personal agents as an unbundler. They could reduce the work that keeps me using the same service out of habit.

But the work has to move somewhere. If the agent knows my budget, dietary restrictions, preferred brands, and the substitutions I rejected last time, replacing the agent may be harder than replacing the store.

**Personal agents could make services more interchangeable while making the agent provider less interchangeable.**

### What an agent actually unbundles

The products are starting to arrive. Grok Bot launched in August, Meta introduced Muse in September, and OpenAI announced dots on September 29. Their announcements describe persistent agents with their own computers, memory, and the ability to work across existing apps.[^grok][^muse][^dots] Those are product claims; how reliably they complete real tasks is a separate question.

The promise is easy to understand. I describe what I want, and the agent works out which services to use and how to use them.

For a routine purchase, that could weaken one source of customer loyalty: the effort of doing things differently. I would still need accounts, payment authorization, and somewhere that delivers to my address. But I might spend less time learning another interface or comparing its options.

This is most plausible where the underlying services are reasonably substitutable. Comparing two retailers is different from moving a social network whose value depends on where everyone else is. An agent can remember my contacts without making those people move to another platform.

The first effect may therefore be narrower than the phrase "ultimate unbundler" suggests. Agents could make more services available through one interface. Whether that gives users more freedom depends on who controls the interface and which services it can actually reach.

### Amazon and Shopify see different tradeoffs

In September, Amazon blocked Meta's Muse from completing shopping tasks on its site, according to reporting by Android Headlines citing GeekWire. Amazon said Meta lacked authorization and that Muse did not properly identify itself, raising concerns about credentials and access to customer accounts. Meta described protected credential storage that allows its agent to use passwords without seeing them.[^amazon]

Around the same time, Shopify announced a partnership to enable Muse checkout through Shop Pay on Shopify stores.[^shopify]

The contrast makes sense when I look at what each company sells.

Amazon operates a shopping destination. Discovery, recommendations, advertising, checkout, and fulfillment are connected parts of that experience. If another company's agent chooses the product and seller, Amazon may still receive an order, but it has fewer opportunities to influence the purchase or bring the shopper back to its interface. The agent could also compare Amazon with other retailers before deciding where to buy.

Shopify supplies infrastructure to merchants that want orders from many places. An agent can become another route to those merchants, increasing sales and use of checkout services even when the shopper starts elsewhere. Shopify has an incentive to make its infrastructure useful to agents.

That difference in business structure helps explain the responses. It does not establish Amazon's private motives, or make Shopify disinterested in controlling an important part of commerce. Both companies benefit when more transactions depend on their systems.

Amazon's own **Buy for Me** feature makes the distinction clearer. It uses an agent to buy selected products from other brands' websites while the customer stays in the Amazon app. Amazon says brands can choose whether to participate, and those brands handle delivery, returns, and customer service.[^buyforme]

The feature broadens the available suppliers while keeping Amazon as the place where the shopper starts and confirms the purchase. Acting across services can strengthen the company providing the interface.

The permission arrangements matter too. An identified agent with an agreed checkout integration can present different risks from an agent using an existing account through a browser. Amazon's commercial incentives and its security concerns can both be real. Supporting interoperability means working out how outside agents can operate safely and how responsibility for mistakes is assigned.

Amazon also publishes crawler restrictions covering several OpenAI bots, including ChatGPT-User and OAI-SearchBot.[^robots] Those access policies are distinct from the Muse shopping restriction. Search, retrieval, and the authority to make a purchase are different kinds of access, and a policy file alone does not tell us how every restriction is enforced.

### Apple's case for integration

Apple illustrates another way an agent could develop: deep integration with the devices and systems people already use.

Its Siri AI announcement describes an assistant that can draw on personal messages, email, photos, and onscreen context. Apple also describes third-party app support, with App Intents giving developers a route to expose their apps' capabilities.[^siri][^intents] The announcement set out this architecture with a phased rollout.

A supported Apple device and participating third-party apps can form part of that experience. Owning every Apple product is not a prerequisite. The ecosystem advantage comes from how much useful context and functionality are available through Apple's integration framework.

That can produce a better assistant. A request involving a message, a calendar entry, and a photo is easier to complete when the system already knows how to access each of them and manage the permissions. Consistent identity and privacy controls have value.

It can also increase dependence on the platform. The assistant becomes more useful as more of a person's activity happens within the systems it can access reliably. Moving elsewhere may mean losing some of that convenience.

The practical tradeoff is between the benefits of tight integration and the ability to carry a working assistant across providers. Users may rationally choose integration, especially if it completes tasks more reliably.

### Access has been contested before

The argument over using a chosen tool with an existing service has a long history.

In 1968, the FCC's Carterfone decision rejected AT&T's blanket restriction on customer-supplied interconnecting devices that did not harm the telephone network. AT&T had argued that responsibility for the system required control over the equipment connected to it. The Commission distinguished legitimate protection of the network from restrictions covering harmless devices.[^carterfone]

The useful parallel is the customer's interest in making a service more useful through a tool of their own choosing. A technically capable tool can remain unusable until access rules change.

There are limits to the analogy. Telephone carriers operated under a particular regulatory regime, and the decision established that the equipment at issue was harmless. It does not establish a retailer's obligation to admit an AI agent, or settle whether that agent's behavior is safe.

But it does suggest a question for access policy: what conditions would let a customer use an outside agent without harming the service or other users? Clear identification, limited permissions, and responsibility for errors could turn some disputes into solvable integration problems.

### The agent can become the bundle

Even if access improves, the agent introduces a dependency of its own.

Imagine replacing an agent after six months. A new one might need to learn which substitutions I accept, how I write emails, when I want approval, and what happened in an unfinished project. It would need access to the relevant accounts and an understanding of the boundaries I set.

Some of that information could be exported. Some might live in routines, files, memory tied to a model, or permissions that need to be granted again. A transcript is useful, but it is not necessarily a portable working relationship.

Whether that becomes durable lock-in depends on the products' export and compatibility choices. The important distinction is between being able to download my data and being able to use it effectively with another agent.

This is why an agent's ability to reach many services alone does not persuade me that it is liberating. It might make retailers compete harder while concentrating more of my daily life around one agent provider.

Muse itself complicates the picture. It is a Meta product, and its launch examples connect Instagram and WhatsApp with tasks across external services.[^muse] Existing distribution and the ability to work across services can reinforce each other. Large platforms can challenge another company's boundaries while increasing the value of their own products.

### Comparison creates its own gatekeepers

Online travel offers a counterexample to the idea that easier comparison necessarily disperses power.

Booking sites made it easier to compare accommodation providers. They also became important intermediaries between travelers and those providers. In May 2024, the European Commission designated Booking as a gatekeeper for Booking.com under the Digital Markets Act.[^booking]

A service can improve access to suppliers and become a powerful point of control over which supplier gets chosen. That possibility deserves attention with agents, because the selection may happen with very little user involvement.

The business model matters. A subscription paid by the user creates different incentives from merchant transaction fees or referral payments. None guarantees good or bad recommendations, but each changes what the operator gets rewarded for.

OpenAI's 2025 Instant Checkout announcement described a fee paid by merchants on completed purchases. It also stated that product results were organic and unsponsored, while checkout availability was one consideration when ranking merchants offering the same product.[^checkout]

Those were the company's stated policies at launch. They illustrate how several decisions can sit inside what looks to the user like a single recommendation: which product to choose, which merchant to use, and which checkout route is available.

I would want an agent to make those tradeoffs understandable. An integrated checkout might be more convenient or reliable. The user should be able to tell when that convenience affected the choice.

### Where I could be wrong

I may be overestimating how much loyalty comes from the effort of switching. Fast delivery, predictable returns, subscriptions, and trusted support can make a provider worth choosing repeatedly. An agent optimizing my actual preferences may keep sending me to the same store.

I may also be overestimating how quickly people will delegate. An agent that needs frequent corrections can cost more attention than it saves. Recommendations may change before purchases, and purchases before tasks involving health, money, or relationships.

Finally, I may be underestimating the integrated systems. If native assistants have better context and complete tasks more reliably, users could move more activity into one ecosystem. The technology that promises to reduce dependence on individual apps could increase dependence on the platform connecting them.

### What I expect to happen

My working expectation is uneven, negotiated access. Some services will welcome agents as a distribution channel. Others will require partnerships, impose conditions, or restrict access. Native assistants will have advantages wherever useful context and permissions are easier to obtain inside an existing platform.

The Shopify–Muse partnership and OpenAI's commerce protocol offer early examples of integration through agreed transaction routes.[^shopify][^checkout] Standards can reduce the engineering work of connecting an agent to a merchant. The remaining questions include access terms, liability, fees, recommendations, and portability of personal context.

I would watch three things:

- **Useful portability:** whether a person can move memory and routines to another agent without rebuilding months of context.
- **Authorized access:** whether outside agents can complete tasks under clear conditions across a broad range of services.
- **Recommendation incentives:** whether users can understand how fees, partnerships, and integration affect the agent's choices.

Together, those will say more about user freedom than how many apps an agent can connect to.

Personal agents could reduce the work that makes me stay with a service out of habit. They could also create an intermediary I depend on for an increasing share of my decisions.

I expect personal agents to make more services replaceable. I am less sure they will make the agent itself replaceable.

[^grok]: SpaceXAI, ["Introducing Grok Bot"](https://x.ai/news/introducing-grok-bot), August 11, 2026. Product announcement; descriptions of autonomy and reliability are vendor claims.
[^muse]: Meta, ["Introducing Muse: The World's First Personal AI Agent Built for Everyone"](https://about.fb.com/news/2026/09/introducing-muse-personal-ai-agent/), September 8, 2026, updated September 30. Product, memory, credential-protection, and integration descriptions are Meta's claims.
[^dots]: OpenAI, ["Introducing dots"](https://openai.com/index/introducing-dots/), September 29, 2026. Announcement of a rollout beginning for eligible paid plans.
[^amazon]: Jean Leon, Android Headlines, ["Amazon Blocks Meta's Muse AI Agent from Shopping on Its Storefront"](https://www.androidheadlines.com/2026/09/amazon-blocks-meta-muse-ai-agent-shopping.html), September 22, 2026, reporting on GeekWire's coverage. Amazon's authorization and security concerns and Meta's response are attributed claims.
[^shopify]: PYMNTS, ["Shopify Brings Shop Pay Checkout Solution to Meta's Muse AI Agent"](https://www.pymnts.com/commerce/ecommerce/2026/shopify-brings-shop-pay-checkout-solution-to-metas-muse-ai-agent/), September 22, 2026, reporting the September 21 partnership announcement.
[^buyforme]: Amazon, ["Amazon's new 'Buy for Me' feature helps customers find and buy products from other brands' sites"](https://www.aboutamazon.com/news/retail/amazon-shopping-app-buy-for-me-brands). Primary description of the beta feature and participation arrangements, accessed October 2, 2026.
[^robots]: Amazon, [robots.txt](https://www.amazon.com/robots.txt), observed October 2, 2026. The file listed `Disallow: /` for GPTBot, ChatGPT-User, and OAI-SearchBot. This mutable policy file is distinct from evidence of technical enforcement or shopping-agent permissions.
[^siri]: Apple, ["Apple introduces Siri AI, a profoundly more capable and personal assistant"](https://www.apple.com/newsroom/2026/06/apple-introduces-siri-ai-a-profoundly-more-capable-and-personal-assistant/), June 2026. Announcement describing personal context, third-party app support, supported devices, and planned beta availability.
[^intents]: Apple, ["Apple aids app development with new intelligence frameworks and advanced tools"](https://www.apple.com/newsroom/2026/06/apple-aids-app-development-with-new-intelligence-frameworks-and-advanced-tools/), June 2026. Description of App Intents integration with Siri AI.
[^carterfone]: FCC, ["Use of the Carterfone Device in Message Toll Telephone Service," 13 FCC 2d 420](https://www.nicholasjohnson.org/FCCOps/1968/13F2-420.html), June 26, 1968. Decision text reproduced on former FCC Commissioner Nicholas Johnson's website.
[^booking]: European Commission, ["Commission designates Booking as a gatekeeper and opens a market investigation into X"](https://digital-markets-act.ec.europa.eu/commission-designates-booking-gatekeeper-and-opens-market-investigation-x-2024-05-13_en), May 13, 2024.
[^checkout]: OpenAI, ["Buy it in ChatGPT: Instant Checkout and the Agentic Commerce Protocol"](https://openai.com/index/buy-it-in-chatgpt/), September 29, 2025. The fee and ranking descriptions above refer to the stated policies at that launch.
