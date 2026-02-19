---
name: spec-estimator
description: Takes a technical spec and produces a comprehensive effort estimate calibrated for AI-assisted development (2026). Generates a research prompt that analyzes the spec, identifies complexity drivers, researches current tooling/libraries, and outputs a detailed estimate in both legacy human-hours and realistic AI-assisted hours — including parallel agent strategies.
---

# Spec Estimator

A skill that reads a technical specification and produces a rigorously researched effort estimate for building it — calibrated for how software actually gets built in 2026: with AI coding agents, parallel execution, and spec-driven workflows.

## Why This Exists

Traditional estimation frameworks (story points, t-shirt sizing, planning poker) were designed for a world where humans write every line of code sequentially. That world is gone.

In 2026, a single developer with AI agents can mass-produce 21,000+ lines of working, deployed code in 5 hours. A spec that would have taken 200 human-hours in 2023 now takes 4-8 hours of human orchestration time. But most estimation tools haven't caught up.

This skill bridges that gap. It takes a spec, researches what's actually involved, and produces an estimate that accounts for:

- AI agent capabilities (what can be fully delegated vs. what needs human judgment)
- Parallel agent execution (what tasks can run simultaneously)
- Known friction patterns (SDK issues, wrong-approach false starts, integration complexity)
- The real bottleneck: human decision-making and review, not code production

## What This Skill Does

Given a spec (any format — markdown, doc, PDF, plain text), this skill:

1. **Parses the spec** into discrete functional areas and deliverables
2. **Classifies each area** by complexity type (not just effort)
3. **Researches** current libraries, SDKs, frameworks, and known pitfalls relevant to the spec
4. **Produces a research prompt** that can be run via deep research to gather everything needed for accurate estimation
5. **Outputs a structured estimate** covering both legacy and AI-assisted timelines

## How To Use This Skill

### Quick Start

```
/estimate path/to/spec.md
```

Or in conversation:

```
Read the spec at docs/my-feature-spec.md and estimate the effort to build it.
```

### What You Get Back

A markdown report with:

- **Spec decomposition** — every functional area broken into estimable units
- **Complexity classification** — each unit tagged by what makes it hard
- **Research prompt** — a ready-to-run deep research prompt covering all unknowns
- **Three-column estimate** — Legacy (2023 human-solo), AI-Assisted (single agent), AI-Parallel (multi-agent orchestration)
- **Risk map** — where estimation confidence is low and why
- **Recommended agent strategy** — how to decompose work across parallel agents

---

## The Estimation Process

### Phase 1: Spec Decomposition

Read the spec and extract every discrete functional area. For each area, capture:

- **What it does** (1-2 sentences)
- **Inputs and outputs** (data flowing in and out)
- **Dependencies** (what must exist before this can be built)
- **Integration surface** (what other areas does this touch)

Group areas into:

| Category | Description | Examples |
|----------|-------------|----------|
| **Core Logic** | The hard algorithmic or architectural work | Layout engines, data pipelines, state machines |
| **Integration** | Connecting systems, APIs, SDKs | Database queries, third-party API calls, auth flows |
| **UI/UX** | Visual components and interaction | Components, animations, responsive layouts |
| **Infrastructure** | Deployment, config, environments | Docker, CI/CD, environment setup |
| **Data** | Schema design, migrations, transformations | Database schemas, ETL, data format conversions |
| **Polish** | Edge cases, error handling, testing | Validation, error states, cross-browser, tests |

### Phase 2: Complexity Classification

For each functional area, classify the **type** of complexity. This matters because AI agents handle different complexity types very differently.

| Complexity Type | What It Means | AI Delegation Score | Examples |
|----------------|---------------|---------------------|----------|
| **Boilerplate** | Well-known patterns, standard implementations | 95% — fully delegate | CRUD endpoints, form components, config files |
| **Library Integration** | Using existing packages with known APIs | 80% — delegate with SDK-read-first | Database ORMs, charting libraries, auth providers |
| **Novel SDK** | Using lesser-known or rapidly-changing SDKs | 50% — delegate but expect friction rounds | TurboPuffer, niche APIs, bleeding-edge packages |
| **Algorithmic** | Custom logic requiring domain understanding | 60% — delegate with detailed spec, review output | Layout algorithms, scoring systems, search ranking |
| **Architectural** | System design decisions with tradeoffs | 30% — human decides, AI implements decision | Service boundaries, data flow design, caching strategy |
| **Creative/UX** | Subjective quality requiring human judgment | 40% — AI drafts, human iterates | Visual design, interaction feel, copy/tone |
| **Integration Glue** | Making independently-built pieces work together | 50% — high friction, needs human debugging | Cross-service data flow, state synchronization |
| **Unknown/Research** | Requires investigation before estimation is possible | 0% until researched — this is what the research prompt handles | "Does X even support Y?", "What's the best approach for Z?" |

### Phase 3: Research Prompt Generation

This is the core output of the skill. For every area classified as Novel SDK, Unknown/Research, or anything with low estimation confidence, generate a structured research prompt.

The research prompt MUST:

1. **List every specific technical question** that must be answered before estimating
2. **Identify libraries/SDKs** that need capability verification
3. **Flag architectural decisions** that affect multiple areas
4. **Ask for real-world benchmarks** — "has anyone built X with Y? how long did it take?"
5. **Check for existing solutions** — "is there a library that already does 80% of this?"

#### Research Prompt Template

Generate the research prompt in this format:

```markdown
# Estimation Research: [Spec Title]

## Context
[2-3 sentence summary of what's being built and the tech stack]

## Research Questions

### Libraries & SDKs
For each technology referenced or implied by the spec:
- [ ] What is the current stable version of [library]?
- [ ] Does [library] support [specific capability needed]?
- [ ] What are known gotchas, breaking changes, or API instability issues?
- [ ] Are there better alternatives that have emerged recently?

### Architecture
- [ ] [Specific architectural question from the spec]
- [ ] What is the standard/recommended approach for [pattern] in [framework]?
- [ ] Are there existing open-source implementations of similar systems?

### Integration Points
- [ ] How does [System A] connect to [System B] in the current ecosystem?
- [ ] What middleware/adapters exist for [specific integration]?
- [ ] What are common failure modes in [integration pattern]?

### Unknowns
- [ ] [Any area where the spec is ambiguous or implies capabilities that need verification]

### Benchmarks
- [ ] Find real-world examples of similar projects — what was the actual build time?
- [ ] What's the typical iteration count for [complex area] when using AI coding agents?
- [ ] What parallel agent strategies have worked for [similar project type]?

## Estimation-Critical Findings
[This section gets filled in by the research — it should contain the facts that most affect the estimate]
```

### Phase 4: Effort Estimation

After research (or with best-available knowledge if skipping research), produce the estimate.

#### Estimation Model

For EACH functional area, estimate three columns:

| Column | What It Represents | How To Calculate |
|--------|-------------------|------------------|
| **Legacy (2023)** | Senior developer, no AI, writing all code | Traditional estimation — complexity × uncertainty × integration overhead |
| **AI-Single** | One developer + one AI agent (e.g., Claude Code) | Legacy ÷ AI delegation score, plus friction allowance |
| **AI-Parallel** | One developer orchestrating multiple simultaneous agents | AI-Single ÷ parallelism factor, but add orchestration overhead |

#### Friction Allowances

Based on empirical data from high-volume AI-assisted development:

| Friction Type | Frequency | Time Cost | Mitigation |
|--------------|-----------|-----------|------------|
| Wrong approach on first attempt | ~35% of tasks | 2-3 extra rounds | Better spec scoping, CLAUDE.md rules |
| SDK API signature errors | ~25% of integration tasks | 1-2 extra rounds | SDK-read-first pattern |
| Misunderstood request | ~20% of tasks | 1-2 extra rounds | Explicit layer/component framing |
| Integration bugs at boundaries | ~30% of multi-system tasks | 2-4 extra rounds | Interface contracts, integration tests |
| Scope creep by agent | ~10% of tasks | 1 round to correct | Strict scope rules in CLAUDE.md |

Apply friction allowances multiplicatively:
```
AI-Adjusted Hours = Base Hours × (1 + Σ applicable friction rates)
```

#### Parallelism Factors

Not everything can run in parallel. Use these guidelines:

| Parallelizable? | Condition | Factor |
|----------------|-----------|--------|
| **Fully parallel** | No shared state, independent files, clear boundaries | N agents → ~1/N time |
| **Partially parallel** | Some shared interfaces, sequential dependencies in parts | N agents → ~1/(N×0.6) time |
| **Sequential only** | Deep dependency chains, shared state, architectural decisions | No parallelism benefit |

#### Output Format

```markdown
# Effort Estimate: [Spec Title]

## Summary

| Metric | Legacy (2023) | AI-Single | AI-Parallel |
|--------|:------------:|:---------:|:-----------:|
| **Total Hours** | X | Y | Z |
| **Calendar Days** (assuming 6hr productive/day) | A | B | C |
| **Human Decision Hours** | — | H₁ | H₂ |
| **Agent Execution Hours** | — | E₁ | E₂ |

## Decomposition

### [Functional Area 1]
- **What:** [description]
- **Complexity Type:** [from classification]
- **Legacy:** X hrs — [reasoning]
- **AI-Single:** Y hrs — [reasoning, delegation score, friction allowance]
- **AI-Parallel:** Z hrs — [reasoning, parallelism factor]
- **Confidence:** High/Medium/Low — [why]

[repeat for each area]

## Risk Map

| Risk | Impact on Estimate | Likelihood | Mitigation |
|------|-------------------|------------|------------|
| [risk] | +X hrs | High/Med/Low | [action] |

## Recommended Agent Strategy

### Parallel Workstreams
[How to decompose across agents — which areas are independent, 
what order to tackle sequential dependencies, where to set up 
integration contracts between parallel streams]

### Suggested Execution Plan
[Concrete plan: "Agent 1 builds X while Agent 2 builds Y, 
then Agent 3 handles integration, human reviews at checkpoints A, B, C"]

## Research Gaps
[Anything that couldn't be estimated confidently without further research,
with the generated research prompt for follow-up]
```

---

## Calibration Notes

These calibration benchmarks come from real-world AI-assisted development data:

| Project Type | Legacy Estimate | Actual AI-Assisted Time | Speedup |
|-------------|:-----------:|:-------------------:|:-------:|
| Custom mind map control (GoJS replacement) | ~200 hrs | ~8 hrs human time | 25x |
| Full-stack web app (FastAPI + Next.js + Redis) | ~120 hrs | ~15 hrs human time | 8x |
| Data pipeline (Postgres + vector DB + ETL) | ~80 hrs | ~12 hrs human time | 7x |
| SDK integration with novel API | ~40 hrs | ~10 hrs human time | 4x |

Key insight: **Speedup is NOT uniform.** Boilerplate-heavy projects see 20-25x. Integration-heavy and research-heavy projects see 4-8x. Architectural projects see 3-5x. The skill must account for the MIX of work in the spec, not apply a flat multiplier.

### What Makes Estimates Wrong

The most common estimation failures in AI-assisted development:

1. **Underestimating integration complexity** — individual pieces build fast, gluing them together still takes human judgment
2. **Ignoring SDK friction** — AI agents confidently write wrong API calls for lesser-known libraries
3. **Assuming flat speedup** — applying a single multiplier instead of varying by complexity type
4. **Forgetting human review time** — code gets written fast but still needs to be understood and validated
5. **Not accounting for false starts** — agents sometimes go down completely wrong paths requiring restart

---

## Example Usage

### Input
```
Estimate the effort for this spec: docs/real-time-dashboard-spec.md
Tech stack: Next.js, FastAPI, WebSockets, PostgreSQL, Redis
```

### Output
The skill produces:
1. A decomposition of the dashboard spec into ~8-12 functional areas
2. A research prompt covering WebSocket scaling, real-time charting libraries, and Redis pub/sub patterns
3. A three-column estimate showing something like:
   - Legacy: ~160 hrs
   - AI-Single: ~20 hrs human orchestration
   - AI-Parallel (3 agents): ~10 hrs human orchestration
4. A risk map flagging WebSocket state management as the highest-risk area
5. A concrete parallel agent execution plan

---

## Configuration

The skill works with any spec format. It adapts to the tech stack mentioned in the spec or provided by the user.

If no tech stack is specified, the skill will ask before estimating, because technology choices dramatically affect the estimate (building a custom layout engine in raw Canvas vs. using d3-hierarchy is a 3x difference).

The skill defaults to producing output in markdown (`.md`). It will never default to `.docx` unless explicitly asked.

---

## License

MIT — open source, use freely, contribute back.
