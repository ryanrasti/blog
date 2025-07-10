---
title: "Building with Elixir for Three Years: A Production Retrospective"
description: "Real lessons from building a warehouse management system from zero to production MVP with Elixir."
pubDate: "Jul 10 2025"
heroImage: "/src/assets/elixir-wms-optimized.jpg"
---

I recently responded to a [Hacker News thread](https://news.ycombinator.com/item?id=44495879) about Elixir's benefits. As CTO of a startup that built with Elixir for two years before deploying to production in our third, I wanted to share our actual experience versus the marketing promises.

We built a warehouse management system from scratch — complete with multi-warehouse inventory tracking, order routing, pick/pack workflows, and dozens of e-commerce integrations. We successfully went live with our MVP and first customers, and have since fulfilled over 100,000 orders. Here's what worked, what didn't, and what I'd do differently.

## The BEAM: Living Up to the Hype

The Erlang VM delivered on its core promise. Scalability, concurrency, and resilience "just worked." No orchestration needed, no complex tooling, no surprises. You avoid the typical distributed systems mess — no gluing together Redis for queues, Kubernetes for orchestration, and a dozen other tools. Everything you need comes built-in. When you are running a lean team, this reduced surface area means you can iterate faster with fewer things that can break.

## The Ecosystem: Depth Over Breadth

The ecosystem is hit or miss. I would characterize it as having exceptional depth where it matters but frustrating gaps elsewhere.

[Phoenix](https://phoenixframework.org/) and [Oban](https://hexdocs.pm/oban/Oban.html) were fantastic — world-class libraries that did exactly what we needed. 

[Ecto](https://hexdocs.pm/ecto/Ecto.html) deserves special mention. This query builder opened my eyes to what is possible beyond raw SQL. It saved us countless times with its ability to dynamically compose queries — in that sense, it is even more powerful than raw SQL. The biggest drawback? Most extensions require writing macros, which are hard to grasp when you are picking up Elixir for the first time.

Step outside that golden path and you will hit walls fast. No good libraries for parsing shell commands. Shopify integration libraries were either missing or unmaintained. When you venture beyond the core ecosystem, be ready to build it yourself.

## Phoenix: The Double-Edged Sword

Phoenix and [LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html) promise to eliminate the split frontend/backend pain that plagues so many teams. No more maintaining separate frontends. No more coordinating across teams. No more API versioning headaches. For rapid iteration, this sounds transformative.

We started with React from day one. LiveView could handle simple interactions beautifully, but we knew it could not deliver the complex UI/UX flows our warehouse management system demanded. We ended up with the exact split-stack complexity Phoenix aims to solve. Looking back, a full-stack Node.js framework would have simplified our client/server interaction considerably.

## Developer Experience: The Achilles' Heel

I love functional programming, and Elixir's standard library is thoughtfully designed. The developer experience fell short compared to TypeScript in ways that compound over time.

**Autocomplete was a coin flip.** Half the time [ElixirLS](https://github.com/elixir-lsp/elixir-ls) simply did not work — either too slow or completely broken. When it did work, the static analysis was not sophisticated enough to provide genuinely useful suggestions. In a 300K line codebase, you need autocomplete just to remember function names and schema attributes. Without it, you are constantly grepping through files.

**Compile times broke my flow.** Our 300K line codebase took 10 seconds to recompile after a one-line change, plus a few more seconds for tests. We used GraphQL to get typing between backend and frontend — crucial for type safety and developer experience, but it came at a steep cost. After several hundred thousand lines of code, the full cycle became painful: recompile Elixir, then regenerate types — easily 20 seconds on beefy hardware for a single-line change. VS Code often choked on the huge type definition changes, requiring a full restart to make IntelliSense happy again. Compare that to the instant feedback loop of [Vite](https://vitejs.dev/). Those 20-second turnarounds might sound trivial, but they shatter concentration and compound into hours of lost productivity.

## The Unexpected Wins

Despite the challenges, Elixir delivered some surprising victories.

**Talent quality was exceptional.** Every Elixir developer we interviewed demonstrated deep technical knowledge and pragmatic problem-solving skills. The community self-selects for engineers who choose tools for sound technical reasons, not resume padding or trend chasing. Elixir also made our roles more appealing — talented engineers were excited to work with the language, giving us an edge in recruiting.

**Oban saved us repeatedly.** We leaned on it heavily for background jobs, and I honestly cannot imagine building our system without it. Rock solid.

**Remote IEx got us through MVP.** Being able to debug production issues in real-time via remote console access was absolutely clutch for our launch. Not something you would want to rely on forever, but invaluable when you need it.

## Deployment: More Complex Than Expected

Elixir's distributed nature, while powerful, created unexpected deployment challenges. We started on Fly.io but their repeated instability and lack of managed Postgres forced us to look elsewhere. Cloud Run seemed perfect until we discovered it did not support the peer-to-peer ports Elixir needed for clustering.

We ended up running virtual machines with [NixOS](https://nixos.org/) to manage the overhead. The alternative was Kubernetes, but even managed Kubernetes felt like overkill for our needs. NixOS made VM management tolerable, but it was still more operational complexity than we had budgeted for.

## The Verdict

Elixir shines as the right tool for specific problems. For us building a full-stack application, the experience was decidedly mixed.

Part of this challenge is structural — JavaScript owns the frontend, giving JavaScript frameworks an inherent full-stack advantage that is hard to overcome. There is also significant room for Elixir to improve its developer experience, particularly around tooling and compile times. I am not alone in this assessment — [James Russo from Brex](https://boredhacking.com/areas-of-improvement-for-elixir/) documented similar pain points after running Elixir at scale there.

Would I choose Elixir again? It depends entirely on what you're building. Need true concurrency and fault tolerance for a distributed system? Absolutely. Building a full-stack application where developer velocity is paramount? I would carefully evaluate other options first.

The key lesson after three years is actually an old lesson: use the right tool for the job. Elixir excels at certain challenges — distributed systems, real-time features, fault tolerance. Force it into the wrong context, and you will fight it the entire way.

Most of the pain points I encountered are technical problems with technical solutions. Better tooling, faster compilation, improved static analysis — these are solvable challenges. The Elixir community is talented and pragmatic. I hope they tackle these issues, because the core technology is genuinely impressive.

## The Future: The AI Wild Card

The biggest unknown is how AI will reshape the landscape. I see two competing possibilities — both entirely plausible yet leading to radically different outcomes:

1. **AI accelerates Elixir's evolution.** LLMs could help the smaller Elixir community punch above its weight — addressing tooling gaps faster, improving documentation, and letting the language's strengths shine through.

2. **AI drives consolidation around TypeScript.** Every new project becomes full-stack Next.js. Old projects get rewritten. The AI hive mind converges on "best practices" and every LLM regurgitates the same stack recommendation. Alternative approaches get buried under the weight of generated sameness.

My bet? The language with the best feedback loop to LLMs wins. TypeScript has a huge advantage here. Static types give LLMs clear signals about correctness. At our startup, we reached a point where TypeScript eliminated virtually all runtime type errors — no null pointer exceptions, no frontend crashes from type mismatches. I have only heard similar stories from niche functional languages like Elm.

Ironically, functional programming should be perfect for LLMs. When there is no global mutable state, you need much less context to understand what code does. Elixir has this advantage. TypeScript does not.

One last thought: the alternatives are not standing still. TypeScript already has the superior tooling and instant feedback loops. New tools like [tsgo](https://devblogs.microsoft.com/typescript/typescript-native-port/) promise 10x faster performance (we've seen 5x improvements in our codebase). Meanwhile, TypeScript keeps adding features that chip away at functional programming's advantages — better immutability support, stricter null checking, even pattern matching proposals.

Elixir's window to catch up might be closing. The race is on.