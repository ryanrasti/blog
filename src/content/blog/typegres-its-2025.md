---
title: "It's 2025. Stop Bringing a Golf Cart to a Formula 1 Race"
subtitle: "Typegres: A Unified Theory of Data Access"
description: "Why I'm building a PostgreSQL-first data layer for TypeScript"
pubDate: "Oct 7 2025"
heroImage: "/src/assets/typegres-its-2025.jpg"
---

# 2025: The F1 of Developer Velocity

If you’re building an app, you’ve signed up for a race. The winning metric isn’t runtime performance – it’s **developer velocity**. Your lap time is how fast you can ship a feature.

In 2025, the world has converged on an elite setup:

- Your world-class **driver** is **TypeScript** – your FE and BE stack in one and the most advanced type system to gain mass adoption
- You're on the **F1 racetrack** of databases: **Postgres** – the de-facto platform for serious applications.

So what’s the bottleneck? It’s not the driver or the track. It’s the **data layer** – the car you bring to the race. Every feature you ship is, at its core, just a wrapper around a SQL query or mutation.

You have an elite driver and a world-class circuit. You should be winning. So why does it feel like your product velocity is stuck at 25 mph?

**Because your data layer thinks it’s playing Mario Kart**

# 2025: Mario Kart on F1

Let’s look at the cars we’ve been told we have to use.

<img src="/images/typegres/golf-cart-orm.jpg" alt="Plumber in a Golf Cart with 'ORM' streak" style="float: left; width: 240px; margin: 0 1.5rem 1rem 0;" />

### **The ORM: The Golf Cart**

ORMs got us on the road. They gave us object mapping, a decent developer experience, and migration handling.

But they’re golf carts. You trade the expressive power of SQL for a toy interface that requires yet another bespoke mental model to operate.

**Verdict: You’ll get to the finish line… days after the race is over**

<img src="/images/typegres/jeep-query-builder.jpg" alt="gorilla with tie in off-road suv with no doors; with 'query builder'" style="float: left; width: 240px; margin: 0 1.5rem 1rem 0;" />

### **The Query Builder: The All-Terrain SUV**

The query builder was a big step up. It's versatile and solves the critical problem of dynamic query construction, acting as the macro system SQL was always missing.

But its core principle – **database agnosticism** – forces you into the **lowest common denominator of SQL**. That might have made sense in 2015. But now it's 2025. Postgres won, and it packs over 3000 built-in functions and advanced features that are just sitting there unused. You're preparing for an off-road trip that will never happen while getting lapped on the racetrack every day.

**Verdict: A solid compromise in 2015, a delusional handicap in 2025.**

<img src="/images/typegres/just-use-sql.jpg" alt="image of a tall villain with curly mustache and wrench changing the tire of a rusting 1980s race car" style="float: left; width: 240px; margin: 0 1.5rem 1rem 0;" />

### **"Just use SQL": The DIY 1980s Model**

Let's be clear what "Just use SQL" really means: it's a bait-and-switch. You're not using the full, expressive power of the SQL language. You're forcing your modern, sophisticated driver (TypeScript) to do something primitive: **clumsily stitching together raw SQL strings**.

This is like telling your F1 driver to pull over mid-race, get out, and service the car himself. You've fired the entire pit crew of modern tooling for dynamic composition and ergonomic relation handling.

**Verdict: A world-class driver with a toolbox instead of a steering wheel.**

<img src="/images/typegres/mad-scientists.jpg" alt="image of a cool kid with red vest in a fusion go-cart with a scientist standing by" style="float: left; width: 240px; margin: 0 1.5rem 1rem 0;" />

### **The Mad Scientists: The Nuclear Fusion Go-Carts**

The most ambitious solutions argue the problem is so fundamental, we need a brand new car. PRQL designs a more ergonomic syntax and logic engine. EdgeDB designs a better data model. They are both brilliant, visionary projects.

But this isn't a test run; it's a race. You’re learning a new query language from scratch. You roll into your pit stop and Doc Brown is scratching his head: he doesn’t have the parts you need. You've abandoned an entire proven ecosystem on a hunch.

**Verdict: Maybe it's the future – But you’ve got a race to win today.**

# Typegres: The Race Car

Typegres starts from a single conviction: that you should use the full power of the tools you've already chosen. It’s a **translator, not another abstraction** that takes the best of all worlds without their compromises:

- From **ORMs**, first-class relations
- From **Query Builders**, dynamic composition
- From **Postgres**, every function, operator, and statement – fully typed (no SQL strings!)
- From **PRQL**, composable, fluent API philosophy
- From **EdgeDB**, graph-like querying in a relational model

The key insight: **translating Postgres to Typescript**. Not ditching SQL like ORMs. Not thinking in strings like query builders. Instead, operating at the level of _SQL expressions_ – turning every Postgres type and function into fully-typed, idiomatic TypeScript building blocks.

The result is an expressive, ergonomic API, showcased by its killer feature: class-based models that naturally model computed columns and relations.

```typescript
class Driver {
  performanceScore() {
    // Extend your models with business-logic, allowing you to think
    // conceptually, instead of in terms of raw database columns.
    return this.podiums.divide(this.racesStarted);
  }

  bestLapAt(track: string) {
    // Relations are first-class in Typegres – use them anywhere
    // you’d use a query:
    return LapTime.where((l) => l.driverId.eq(this.id).and(l.track.eq(track)))
      .orderBy((l) => l.time, { asc: true })
      .limit(1);
  }
}

const topDrivers = await Driver.where((d) => d.racesStarted.gt(50))
  .orderBy((d) => d.performanceScore(), { desc: true })
  .limit(10)
  .select((d) => ({
    name: d.name,
    score: d.performanceScore(),
    // Access every Postgres built-in like `age()` as a method
    // with full auto-complete
    careerSpan: d.firstRaceAt.age(),
    bestMonacoLap: d.bestLapAt("Monaco").select((l) => l.time),
  }));
```

And the best part? The entire chain above is fully type-safe, and **compiles to a single, efficient SQL query**.

# Winning the Next Decade: A New Challenger Approaches

The last decade’s data layers optimized for breadth: _works with every database_. The next decade demands depth: _embrace your stack, extract maximum power_.

This maxim is even more important in the age of AI coding. Everyone now has a fleet of robot F1 drivers – Claude Code’s, Cursor’s, and Copilot’s. Instead of one driver, you have dozens racing 24/7. **Choosing the right car is now by far the single, highest leverage choice you can make.**

The paradox is that LLMs are **world-class SQL experts** but need the **tightest leash possible**. They have encyclopedic knowledge and can construct great queries, but one hallucination brings down production.

Typegres provides the perfect balance: the **freedom of unfettered PostgreSQL** with the **constraints of deep, compile-time type checking**. It's the racing machine that lets your AI driver perform at its absolute peak – without sending it into the wall.

TypeScript and Postgres already won.

It's time you brought a race car.

# Calling Early Adopters & Contributors

Typegres is in early preview. The vision and core architecture are solid, but it's not production ready yet.

- **Take it for a spin:** Try it live at [typegres.com/play](https://typegres.com/play)
- **Join the discussion:** Share feedback and ask questions on [GitHub](https://github.com/ryanrasti/typegres).
- **Follow the build**: [Star the repo](https://github.com/ryanrasti/typegres) to watch the progress.
