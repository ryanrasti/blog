---
title: "Building with Nix for Three Years: A Production Retrospective"
description: "Real lessons from using Nix in production - from developer environments to CI/CD, what worked and what didn't."
pubDate: "Aug 23 2025"
heroImage: "/src/assets/nix-production-optimized.jpg"
---

Three years ago, I made a bet on Nix. Not just for developer environments, but for everything - CI/CD, production deployments, even running Postgres locally. With a team of 2-4 engineers building a full-stack Elixir/React application, we needed every productivity edge we could get.

Following my [Elixir retrospective](/blog/elixir-three-years-production), here's what I learned from going all-in on Nix. Spoiler: the promise of "reproducible builds everywhere" is real, but the path there is rougher than any documentation will tell you.

# The Good

1. **Developer environments:** This alone justifies Nix. Every engineer had identical system tools - same `npm`, same `jq`, same `grep` - across Mac and Linux. Zero "works on my machine" issues. We even ran Postgres via Nix locally, matching production exactly. Adding custom scripts to `$PATH`? One line in `flake.nix`. New engineer onboarding? One command: `nix develop`.
1. **Lightning-fast CI iteration:** Our entire CI was just `nix build`. Need a new tool? Add it to `flake.nix`, push, done. No Docker layers, no apt-get dance. Early on, this cut our CI time from 3 minutes to under 1 minute. As we grew, builds that would've taken 30+ minutes stayed under 15, thanks to Nix's aggressive caching. 
1. **True reproducibility:** When production broke with "cannot get bootfile" errors, we reproduced it locally in minutes. The [issue](https://community.fly.io/t/cannot-get-bootfile-was-there-a-change-to-the-machines-etc-hosts-recently/21710)? Fly.io's `/etc/hosts` file got duplicated during abrupt stops, breaking our Elixir release node naming. Because our entire stack ran identically locally - same Elixir, same system tools, same everything - we could definitively rule out dependency issues and focus on the real problem.
1. **Emergency deployment speed:** Week two of our MVP launch, a customer order to Guam was stuck - we'd missed that USPS territories need customs information. With minutes to unblock the order, I deployed the fix directly from my laptop. No waiting for CI. The binary I built locally was byte-for-byte identical to what CI would produce.

# The Bad

1. **Cross-platform builds remain painful:** Mac developers couldn't build Linux production images locally. Yes, there are VM-based and SSH-based builders, but setting them up properly is a weekend project I never had. This meant our Mac developers had to rely on CI for final builds.
1. **Hermetically building a web-app is a minefield:** Making Elixir + React build hermetically required custom forks of `mix2nix` (for GitHub and private repos) and [spoon-feeding `node2nix` npm v2 lockfiles](https://github.com/svanderburg/node2nix/issues/312). We also had to use IFD (Import From Derivation) to make builds truly hermetic, but makes builds slower and harder to debug. The end result was worth it - `nix build` always worked - but getting was its own journey.
1. **The learning ~curve~ cliff is real:** You need to grok derivations, master arcane syntax, and navigate a standard library via GitHub search. I was already sold on the vision, so I persevered. Most engineers take one look at `{ pkgs ? import <nixpkgs> {} }` and "nope" out. 
1. **Tooling from the stone age:** No autocomplete. No inline docs. Want to know how to use a function? grep nixpkgs. Need examples? grep harder.
1. **Self-hosted runners become a job:** To leverage Nix's caching, we ran our own GitHub Actions runners. This cut build times dramatically but became another system to maintain. Not Nix's fault - more like a hidden cost of doing Nix right.

# The Verdict

Would I use Nix again? Absolutely - but differently.

Nix gave us superpowers. Developer environments alone saved us countless hours. Reproducible deployments let us move fast without breaking things (much). When production did break, we could debug locally with confidence.

But Nix also cost us. Not in money, but in time spent fighting tools, maintaining infrastructure, and explaining to new hires why `mkDerivation` isn't a curse word.

My advice? Start small. Use Nix for developer environments first. If that works, gradually expand.

# The Path Forward: What Nix Really Needs

Nix has solved reproducibility. The technology is revolutionary, but adoption is the unsolved problem.

## 1. A Language Developers Already Know

Nix calls itself "JSON with functions." There's another language that describes though. Imagine if Nix expressions were just TypeScript:

```typescript
// Instead of: { pkgs ? import <nixpkgs> {} }:
// We could write:
import { pkgs } from '@nixpkgs/latest'

export default {
  devShell: pkgs.mkShell({
    packages: [pkgs.nodejs, pkgs.postgresql]
  })
}
```

This is not a far-fetched project. I managed a preliminary PoC on this in a couple weekends, but didn't have the time to take it further.

Imagine, with a TS frontend we get full type-checking, amazing auto-complete, and probably instant adoption from millions of developers.

## 2. Rethinking Integration with External Package Managers

Today, Nix rebuilds every language's package manager from scratch: `node2nix`, `mix2nix`, `poetry2nix`. These tools chase a moving target, translating lockfiles and dependency graphs into Nix derivations. The result? Production-critical tooling that's perpetually half-baked and out of date.

Two paths forward:
1. **Trust existing package managers/lockfiles**: Modern package managers like `npm` are already deterministic - their lockfiles guarantee reproducibility. Instead of reimplementing them, Nix could mark certain tool invocations as "hermetically trusted." Yes, this sacrifices some purity, but the trade-off is clear: 1% less pure, 1000% more approachable.
2. **Intercept at the network layer**: Build a deterministic proxy that captures and replays all package downloads (or at least catalogs and checks the checksums of all network requests). Every `npm install` would hit cached artifacts, ensuring reproducibility without touching the package manager itself. It's more complex than trusting external package managers, but preserves Nix's guarantees.

## 3. The "Vercel for Nix" 

We need a platform that makes Nix deployment magical. Not just `nix deploy`, but:

- **Instant rollbacks**: Every deployment is a derivation. Roll back to any previous state in seconds.
- **Preview environments**: Full-stack previews for every PR with byte-for-byte production parity.
- **Built-in secrets**: Integrate with Nix's string interpolation.
- **Global build cache**: Your team shares all build artifacts automatically.
- **Local = Cloud**: `nix deploy --local` gives you the exact same environment as production.
- **CI Integration**: A CI environment that is Nix and cache friendly. 

No Docker. No Kubernetes. Just Linux, systemd, and Nix. We tried to approximate some of this with Pulumi, but imagine if it were an out-of-the-box supported platform.

The edge here is clear: create your custom cloud, with all of your favorite OSS tools, versioned the exact same in prod and in dev. This could spark an OSS-explosion for dev-ops. 

# Why Now? The AI Moment

AI creates a unique opportunity for Nix.

## 1. AI and Nix Are Made for Each Other

Functional programming and reproducibility solve AI's biggest coding challenges:

- **Pure functions = perfect context**: Without side effects, AI can reason about code in isolation. The function signature tells the whole story.
- **Instant iteration with guardrails**: AI makes mistakes fast, but with Nix it can also fix them fast. Every change is reversible, every environment reproducible.
- **No more "works in ChatGPT"**: When AI generates code with Nix, dependencies are guaranteed. The code that runs in testing runs identically in production.

For those who say "just wait for better models" - that's like saying "AI will eventually write web apps in assembly better than in React." Better developer tools matter, especially for AI.

## 2. The Consolidation Window

AI vibe-coding platforms (Cursor, Replit, Lovable) are consolidating how code gets written. They're making technology choices that will define the next decade. Yes, Replit uses Nix for package management, but imagine a platform that goes all-in - where Nix isn't just managing dependencies but defining the entire stack from dev to deploy.

The window is open. The question is: will someone build this Nix-native future, or will we still be grepping nixpkgs in 2030?

