---
name: spec-estimator
description: Takes a technical spec and produces a comprehensive effort estimate calibrated for AI-assisted development (2026). Generates a research prompt that analyzes the spec, identifies complexity drivers, researches current tooling/libraries, and outputs a detailed estimate in both legacy human-hours and realistic AI-assisted hours — including parallel agent strategies.
---

# Spec Estimator

A skill that reads a technical specification and produces a rigorously researched effort estimate for building it — calibrated for how software actually gets built in 2026: with AI coding agents, parallel execution, and spec-driven workflows.

## Why This Exists

Traditional estimation frameworks (story points, t-shirt sizing, planning poker) were designed for a world where humans write every line of code sequentially. That world is gone.

In 2026, a single developer with AI agents can mass-produce thousands of lines of working, deployed code in a single focused session. A spec that would have taken 200+ human-hours in 2023 now takes 5-40 hours of human orchestration time depending on complexity mix. But most estimation tools — and most AI models doing estimation — haven't caught up.

**The core problem:** Without explicit calibration, AI models default to legacy estimation. They see "custom interactive control" and think "200 hours" because their training data reflects 2023 development speed. This skill exists to force estimation through a calibrated framework that reflects how fast software actually gets built with AI agents in 2026.

This skill bridges that gap. It takes a spec, researches what's actually involved, and produces an estimate that accounts for:

- AI agent capabilities (what can be fully delegated vs. what needs human judgment)
- Parallel agent execution (what tasks can run simultaneously)
- Known friction patterns (SDK issues, wrong-approach false starts, integration complexity)
- Spec quality as a force multiplier (a tight spec is worth 1.5-2x speedup by itself)
- The real bottleneck: human decision-making and review, not code production

## What This Skill Does

Given a spec (any format — markdown, doc, PDF, plain text), this skill:

1. **Parses the spec** into discrete functional areas and deliverables
2. **Classifies each area** by complexity type (not just effort)
3. **Assesses spec quality** — clarity, completeness, and scope definition directly affect the estimate
4. **Researches** current libraries, SDKs, frameworks, and known pitfalls relevant to the spec
5. **Produces a research prompt** that can be run via deep research to gather everything needed for accurate estimation
6. **Outputs a structured estimate** covering both legacy and AI-assisted timelines

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

- **Spec quality assessment** — how much the spec itself accelerates or slows development
- **Spec decomposition** — every functional area broken into estimable units
- **Complexity classification** — each unit tagged by what makes it hard
- **Complexity mix profile** — weighted breakdown that determines the blended Factor
- **Research prompt** — a ready-to-run deep research prompt covering all unknowns
- **Three-column estimate** — Legacy (2023 human-solo), AI-Assisted (single agent), AI-Parallel (multi-agent orchestration)
- **Risk map** — where estimation confidence is low and why
- **Recommended agent strategy** — how to decompose work across parallel agents

---

## CRITICAL: Anti-Legacy-Brain Rules

Before producing ANY estimate, the estimator MUST follow these rules. These exist because AI models systematically overestimate effort when not explicitly calibrated.

### Rule 1: Never Estimate the Concept — Estimate the Actual Work

"Build a custom interactive diagramming control" SOUNDS like a 200-hour project. But if the spec says: single root, tree layout, no structural editing, no keyboard nav, no persistence — the ACTUAL work is dramatically smaller. Always estimate the spec as written, not the category it falls into.

### Rule 2: Classify Before Calculating

NEVER produce a total estimate without first classifying every functional area by complexity type. The blended Factor depends entirely on the mix. A project that's 70% boilerplate and 20% algorithmic with 10% integration has a very different Factor than one that's 50% architectural and 30% novel SDK.

### Rule 3: Check the Domain Count

Single-domain projects (all frontend, all backend, all data) are dramatically faster than multi-domain projects. If the spec lives entirely in one domain (e.g., a React component with no backend), the estimate should reflect that there is ZERO cross-domain integration overhead.

### Rule 4: Check What's Explicitly Excluded

Specs that clearly define what's OUT of scope are telling you the project is smaller than it sounds. If a spec excludes keyboard nav, undo/redo, persistence, multi-select, dark mode — that's not future work, that's scope the estimate must NOT include.

### Rule 5: Spec Quality Is a Multiplier

A precise, unambiguous spec with clear scope boundaries, explicit behavioral definitions, and concrete success criteria reduces AI-assisted development time by 1.5-2x compared to a vague spec of the same scope. Factor this into the estimate.

### Rule 6: Sanity Check Against the Complexity Profile

After calculating, sanity check the AI-assisted estimate against the complexity profile:

| Complexity Profile | Typical AI-Assisted Hours (single dev + agent) |
|---|---|
| Small single-domain, boilerplate-heavy (e.g., CRUD app, simple component) | 2-8 hrs |
| Medium single-domain, mixed complexity (e.g., custom UI control, data viz) | 5-15 hrs |
| Medium multi-domain, integration-heavy (e.g., full-stack feature with auth) | 15-40 hrs |
| Large multi-domain, architectural (e.g., full-stack app, microservice) | 30-80 hrs |
| Large research-heavy, novel SDK (e.g., new platform integration) | 40-120 hrs |

If your estimate falls wildly outside these ranges for the matching profile, re-examine your assumptions.

---

## Phase 0: Spec Quality Assessment

Before decomposing the spec, assess its quality. This directly affects the estimate.

### Spec Quality Scorecard

| Dimension | Score | What To Look For |
|-----------|-------|------------------|
| **Behavioral Clarity** | 1-5 | Are interactions described precisely? Or vague ("should feel good")? |
| **Scope Boundaries** | 1-5 | Is there an explicit "out of scope" section? Are boundaries crisp? |
| **Data Contracts** | 1-5 | Are inputs, outputs, and data formats defined? |
| **Success Criteria** | 1-5 | Is there a clear definition of done? |
| **Technical Specificity** | 1-5 | Does the spec name technologies, or leave stack open? |
| **Ambiguity Level** | 1-5 | How many questions would you need to ask before building? (5 = none) |

**Scoring:**

| Average Score | Spec Quality | Impact on Estimate |
|:---:|---|---|
| 4.5-5.0 | Exceptional — build straight from spec | Multiply AI-assisted hours by 0.6 |
| 3.5-4.4 | Good — minor clarifications needed | Multiply AI-assisted hours by 0.8 |
| 2.5-3.4 | Average — significant interpretation required | No adjustment (1.0x) |
| 1.5-2.4 | Poor — more questions than answers | Multiply AI-assisted hours by 1.5 |
| 1.0-1.4 | Unusable — needs rewrite before estimating | Do not estimate. Request spec revision. |

---

## Phase 1: Spec Decomposition

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

---

## Phase 2: Complexity Classification

For each functional area, classify the **type** of complexity. This matters because AI agents handle different complexity types very differently.

| Complexity Type | What It Means | AI Delegation Score | Factor (Speedup vs Legacy) | Examples |
|----------------|---------------|---------------------|---|----------|
| **Boilerplate** | Well-known patterns, standard implementations | 95% — fully delegate | 10-15x | CRUD endpoints, form components, config files |
| **Library Integration** | Using existing packages with known APIs | 80% — delegate with SDK-read-first | 7-10x | Database ORMs, charting libraries, auth providers |
| **Novel SDK** | Using lesser-known or rapidly-changing SDKs | 50% — delegate but expect friction rounds | 2-4x | Niche APIs, bleeding-edge packages, underdocumented tools |
| **Algorithmic** | Custom logic requiring domain understanding | 60% — delegate with detailed spec, review output | 5-8x | Layout algorithms, scoring systems, search ranking |
| **Architectural** | System design decisions with tradeoffs | 30% — human decides, AI implements decision | 2-3x | Service boundaries, data flow design, caching strategy |
| **Creative/UX** | Subjective quality requiring human judgment | 40% — AI drafts, human iterates | 3-5x | Visual design, interaction feel, copy/tone |
| **Integration Glue** | Making independently-built pieces work together | 50% — high friction, needs human debugging | 3-5x | Cross-service data flow, state synchronization |
| **Unknown/Research** | Requires investigation before estimation is possible | 0% until researched — this is what the research prompt handles | 1-2x | "Does X even support Y?", "What's the best approach for Z?" |

### Computing the Blended Factor

After classifying every functional area, compute the project's complexity mix:

```
Blended Factor = Σ (weight_i × Factor_i) for each functional area i
```

Where `weight_i` is the proportion of total legacy effort that area represents.

**Example:** A project that's 60% boilerplate (Factor 12x), 25% algorithmic (Factor 6x), and 15% integration glue (Factor 4x):

```
Blended Factor = (0.60 × 12) + (0.25 × 6) + (0.15 × 4) = 7.2 + 1.5 + 0.6 = 9.3x
```

This means if the legacy estimate is 90 hours, the AI-assisted estimate is ~10 hours.

---

## Phase 3: Research Prompt Generation

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

---

## Phase 4: Effort Estimation

After research (or with best-available knowledge if skipping research), produce the estimate.

### Estimation Model

For EACH functional area, estimate three columns:

| Column | What It Represents | How To Calculate |
|--------|-------------------|------------------|
| **Legacy (2023)** | Senior developer, no AI, writing all code | Traditional estimation — complexity × uncertainty × integration overhead |
| **AI-Single** | One developer + one AI agent (e.g., Claude Code) | Legacy ÷ area-specific Factor, adjusted for spec quality and friction |
| **AI-Parallel** | One developer orchestrating multiple simultaneous agents | AI-Single ÷ parallelism factor, but add orchestration overhead |

### The Formula

```
AI-Single Hours = (Legacy Hours ÷ Area Factor) × Spec Quality Multiplier × (1 + Friction Allowance)
```

Where:
- **Area Factor** comes from the Complexity Classification table (Phase 2)
- **Spec Quality Multiplier** comes from the Spec Quality Assessment (Phase 0)
- **Friction Allowance** is the sum of applicable friction rates below

### Friction Allowances

Based on empirical data from high-volume AI-assisted development:

| Friction Type | Frequency | Time Cost | Mitigation |
|--------------|-----------|-----------|------------|
| Wrong approach on first attempt | ~35% of tasks | +0.15 per affected area | Better spec scoping, CLAUDE.md rules |
| SDK API signature errors | ~25% of integration tasks | +0.10 per affected area | SDK-read-first pattern |
| Misunderstood request | ~20% of tasks | +0.10 per affected area | Explicit layer/component framing |
| Integration bugs at boundaries | ~30% of multi-system tasks | +0.20 per affected area | Interface contracts, integration tests |
| Scope creep by agent | ~10% of tasks | +0.05 per affected area | Strict scope rules in CLAUDE.md |

**Note:** For single-domain projects with no cross-system integration, the "Integration bugs at boundaries" friction does not apply.

### Parallelism Factors

Not everything can run in parallel. Use these guidelines:

| Parallelizable? | Condition | Factor |
|----------------|-----------|--------|
| **Fully parallel** | No shared state, independent files, clear boundaries | N agents → ~1/N time |
| **Partially parallel** | Some shared interfaces, sequential dependencies in parts | N agents → ~1/(N×0.6) time |
| **Sequential only** | Deep dependency chains, shared state, architectural decisions | No parallelism benefit |

### Output Format

```markdown
# Effort Estimate: [Spec Title]

## Spec Quality Assessment

| Dimension | Score | Notes |
|-----------|:-----:|-------|
| Behavioral Clarity | X/5 | [notes] |
| Scope Boundaries | X/5 | [notes] |
| Data Contracts | X/5 | [notes] |
| Success Criteria | X/5 | [notes] |
| Technical Specificity | X/5 | [notes] |
| Ambiguity Level | X/5 | [notes] |
| **Average** | **X/5** | **Spec Quality Multiplier: Xm** |

## Complexity Mix Profile

| Complexity Type | % of Legacy Effort | Factor | Weighted Contribution |
|---|:---:|:---:|:---:|
| Boilerplate | X% | Yx | Z |
| [etc.] | | | |
| **Blended Factor** | | | **Xx** |

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
- **AI-Single:** Y hrs — [reasoning, Factor, spec quality adjustment, friction allowance]
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

## Calibration Data

These calibration benchmarks come from real-world AI-assisted development data (2025-2026):

### Factor by Project Profile

| Project Profile | Complexity Mix | Blended Factor | Example |
|---|---|:---:|---|
| Single-domain, boilerplate-heavy | 70%+ boilerplate, minimal integration | 8-12x | Custom UI control with tight spec, CRUD app, config dashboard |
| Single-domain, algorithmic | 40% algorithmic, 40% boilerplate, 20% creative | 6-8x | Data visualization, search interface, scoring engine |
| Full-stack feature, well-integrated | 30% boilerplate, 30% integration, 20% library, 20% glue | 5-7x | Authenticated feature with DB, API, and frontend |
| Multi-service, integration-heavy | 40% integration glue, 30% library, 20% architectural, 10% boilerplate | 3-5x | Microservice orchestration, multi-API pipeline |
| Research-heavy, novel stack | 30% unknown, 30% novel SDK, 20% architectural, 20% glue | 2-3x | New platform integration, bleeding-edge framework |

### What Drives Factor Up (Faster Than Expected)

| Accelerator | Impact | Why |
|---|---|---|
| Exceptional spec quality (4.5+/5) | +2-3x on top of base Factor | Eliminates the #1 time sink: figuring out what to build |
| Single technology domain | +1.5-2x | Zero context-switching, no cross-domain integration bugs |
| Well-documented libraries only | +1-2x | AI agents produce correct code on first attempt |
| Explicit scope exclusions | +1-1.5x | Prevents scope estimation inflation |
| Prior art exists (solved problem) | +1-2x | AI can reference patterns, not invent solutions |

### What Drives Factor Down (Slower Than Expected)

| Drag | Impact | Why |
|---|---|---|
| Vague or incomplete spec | 0.5-0.7x on base Factor | Human spends time deciding, not building |
| Novel/underdocumented SDKs | 0.4-0.6x | AI hallucinates APIs, multiple false starts |
| Multi-service integration | 0.5-0.7x | Glue code requires human debugging |
| Architectural ambiguity | 0.5-0.6x | Can't delegate decisions, only implementations |
| Rapidly changing dependencies | 0.3-0.5x | SDK docs outdated, breaking changes mid-build |

### Calibration Example

**Project:** Custom interactive tree-based UI control (React, single-domain, client-side only)

**Spec quality:** 4.7/5 (exceptional — precise behaviors, explicit exclusions, clear data contracts, concrete success criteria)

**Complexity mix:**
- Boilerplate (node rendering, panels, controls, import/export): 50% → Factor 12x
- Algorithmic (balanced tree layout engine): 20% → Factor 6x
- Creative/UX (visual polish, interaction feel): 15% → Factor 4x
- Library Integration (canvas/SVG rendering): 15% → Factor 8x

**Blended Factor:** (0.50 × 12) + (0.20 × 6) + (0.15 × 4) + (0.15 × 8) = 6.0 + 1.2 + 0.6 + 1.2 = **9.0x**

**Spec quality adjustment:** 0.6x (exceptional spec)

**Legacy estimate:** 50-60 hrs (senior frontend dev, 2023, no AI)

**AI-assisted estimate:** 60 hrs ÷ 9.0 × 0.6 = **~4 hrs** (aggressive) to **~8 hrs** (with friction buffer)

**Actual observed:** 5-10 hours. ✓

---

### What Makes Estimates Wrong

The most common estimation failures in AI-assisted development:

1. **Legacy-brain default** — AI models estimate at 2023 speed unless explicitly calibrated. This is the #1 failure mode. A model will say "200 hours" for a project that actually takes 8 hours because it's mentally pricing in human typing speed, not AI agent delegation.

2. **Estimating the category, not the spec** — "Custom diagramming control" sounds like a massive project. But if the spec defines a narrow, specific control with explicit exclusions, the actual scope is 10% of what the category implies. Always estimate the spec as written.

3. **Ignoring spec quality** — A tight spec with precise behaviors and explicit scope boundaries is worth 1.5-2x speedup. A vague spec costs 1.5-2x slowdown. This swing factor alone can make estimates wrong by 3-4x.

4. **Applying flat multipliers** — Using a single "AI speedup" number instead of varying by complexity type. A project that's 80% boilerplate is 10x faster. A project that's 80% architectural is 2.5x faster. Flat multipliers miss this entirely.

5. **Underestimating integration complexity** — Individual pieces build fast, gluing them together still takes human judgment.

6. **Ignoring SDK friction** — AI agents confidently write wrong API calls for lesser-known libraries. Budget extra rounds for novel SDKs.

7. **Forgetting human review time** — Code gets written fast but still needs to be understood and validated.

8. **Not accounting for false starts** — Agents sometimes go down completely wrong paths requiring restart. ~35% of tasks need at least one course correction.

9. **Domain count blindness** — A single-domain project (all frontend, or all backend) has fundamentally different overhead than a multi-domain project. Failing to account for this inflates single-domain estimates and deflates multi-domain estimates.

---

## Example Usage

### Input
```
Estimate the effort for this spec: docs/real-time-dashboard-spec.md
Tech stack: Next.js, FastAPI, WebSockets, PostgreSQL, Redis
```

### Output
The skill produces:
1. A spec quality assessment scoring clarity, scope, and completeness
2. A decomposition of the dashboard spec into ~8-12 functional areas
3. A complexity mix profile showing the weighted Factor breakdown
4. A research prompt covering WebSocket scaling, real-time charting libraries, and Redis pub/sub patterns
5. A three-column estimate with the blended Factor applied per-area
6. A risk map flagging the highest-uncertainty areas
7. A concrete parallel agent execution plan
8. Saves the report in the same folder location as the spec but with [specName]-Estimate.md

---

## Configuration

The skill works with any spec format. It adapts to the tech stack mentioned in the spec or provided by the user.

If no tech stack is specified, the skill will ask before estimating, because technology choices dramatically affect the estimate (building a custom layout engine in raw Canvas vs. using an existing library is a 3x difference).

The skill defaults to producing output in markdown (`.md`). It will never default to `.docx` unless explicitly asked.

---

## License

MIT — open source, use freely, contribute back.
