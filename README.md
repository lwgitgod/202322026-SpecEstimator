# Spec Estimator

**Stop estimating like it's 2023.**

A Claude Code skill that takes any technical spec and produces a real effort estimate calibrated for AI-assisted development — including parallel agent strategies, friction allowances, and a deep research prompt to fill knowledge gaps.

Built from real-world data: a spec estimated at 200 human-hours was actually built in 8 hours of human orchestration time with AI agents. This skill accounts for that reality.

---

## What It Does

You give it a spec. It gives you back:

1. **Spec decomposition** — every functional area broken into estimable units
2. **Complexity classification** — each unit tagged by what actually makes it hard for AI agents (not humans)
3. **Research prompt** — a ready-to-run deep research prompt covering all the unknowns that affect the estimate
4. **Three-column estimate** — Legacy (2023 human-solo), AI-Single (one agent), AI-Parallel (multi-agent)
5. **Risk map** — where the estimate is least confident and why
6. **Agent execution plan** — how to split work across parallel agents for fastest delivery

---

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured

### Install via Claude Code (Recommended)

Open Claude Code and run:

```
/install-skill https://github.com/lwgitgod/spec-estimator
```

That's it. The skill is now available in your project.

### Manual Install — Project Level

If you prefer to add it manually or want to customize:

```bash
mkdir -p .claude/skills/spec-estimator
curl -o .claude/skills/spec-estimator/SKILL.md \
  https://raw.githubusercontent.com/lwgitgod/spec-estimator/main/SKILL.md
```

Or clone and symlink:

```bash
git clone https://github.com/lwgitgod/spec-estimator.git
mkdir -p .claude/skills
ln -s $(pwd)/spec-estimator .claude/skills/spec-estimator
```

### Manual Install — Global (All Projects)

To make the skill available across every project on your machine:

```bash
mkdir -p ~/.claude/skills/spec-estimator
curl -o ~/.claude/skills/spec-estimator/SKILL.md \
  https://raw.githubusercontent.com/lwgitgod/spec-estimator/main/SKILL.md
```

### No npm Needed

This is a Claude Code skill — it's just a markdown file that Claude reads. No runtime, no dependencies, no build step, no `node_modules`. It works the moment it's in your `.claude/skills/` directory.

---

## Usage

### Basic — Estimate a Spec

Open Claude Code in your project and run:

```
Estimate the effort for the spec at docs/my-feature-spec.md
```

Claude will read the skill, decompose the spec, and produce a full estimate report.

### With Tech Stack Context

For better estimates, tell it what you're building with:

```
Estimate the effort for docs/my-feature-spec.md
Stack: Next.js, FastAPI, PostgreSQL, TurboPuffer, Redis
```

### Research-First Mode

If your spec has a lot of unknowns (new SDKs, unfamiliar patterns), ask for the research prompt first:

```
Read docs/my-feature-spec.md and generate a research prompt 
for everything I'd need to know before estimating it accurately.
```

Take that research prompt, run it through deep research or web search, then come back:

```
Here are the research findings: [paste results]
Now produce the full estimate for docs/my-feature-spec.md
```

### Compare Approaches

If you're deciding between two architectural approaches:

```
Estimate the effort for docs/my-feature-spec.md under two scenarios:
1. Using d3-hierarchy for the layout engine
2. Writing a custom layout algorithm from scratch
```

### Quick Estimate (Skip Research)

For well-understood specs where you don't need deep research:

```
Quick estimate for docs/my-feature-spec.md — skip research, 
use best-available knowledge. Stack is Python + FastAPI + Postgres.
```

---

## Understanding the Output

### The Three Columns

| Column | What It Means | Who's Working |
|--------|--------------|---------------|
| **Legacy (2023)** | Traditional estimate — one senior dev, no AI | Human writes all code |
| **AI-Single** | One developer + one AI agent | Human orchestrates, AI implements |
| **AI-Parallel** | One developer + multiple simultaneous agents | Human architects, agents execute in parallel |

The **AI-Single** column is your realistic planning number for most work.

The **AI-Parallel** column is achievable if you structure work into independent streams — the estimate includes orchestration overhead.

### Complexity Types

The skill classifies each area by how well AI agents handle it:

| Type | AI Handles It? | Expect Friction? |
|------|:-:|:-:|
| Boilerplate | ✅ 95% | Rarely |
| Library Integration | ✅ 80% | Sometimes (wrong API calls) |
| Novel SDK | ⚠️ 50% | Often (SDK signature guessing) |
| Algorithmic | ⚠️ 60% | Sometimes (needs detailed spec) |
| Architectural | ❌ 30% | Always (human decisions required) |
| Creative/UX | ❌ 40% | Often (subjective iteration) |
| Integration Glue | ⚠️ 50% | Often (cross-system debugging) |

### Friction Allowances

The estimate bakes in real-world friction rates from empirical data:

- **35%** of tasks: AI picks wrong approach on first attempt (+2-3 rounds)
- **25%** of integration tasks: incorrect SDK API calls (+1-2 rounds)
- **20%** of tasks: AI misunderstands the request (+1-2 rounds)

These aren't theoretical — they come from 233 real Claude Code sessions across production projects.

### Confidence Levels

Each area gets a confidence rating:

- **High** — well-understood patterns, proven libraries, clear spec
- **Medium** — some unknowns but estimable within a range
- **Low** — requires research before a confident estimate is possible

Low-confidence areas are exactly what the research prompt is designed to resolve.

---

## Tips for Best Results

### Write Better Specs, Get Better Estimates

The skill is only as good as the spec you feed it. Specs that estimate well:

- List concrete deliverables (not vague goals)
- Specify the tech stack
- Define what's in scope AND what's explicitly out of scope
- Describe data flows and integration points
- Include acceptance criteria

### Pair With Deep Research

The research prompt the skill generates is designed to be run through Claude's deep research or web search. The pattern is:

```
1. Feed spec to skill → get estimate + research prompt
2. Run research prompt → get findings
3. Feed findings back → get refined estimate
```

This two-pass approach catches the unknowns that make estimates wrong.

### Calibrate With Your Own Data

The default calibration benchmarks are based on real projects but they're not YOUR projects. After you've built a few things, update the calibration table in SKILL.md with your actual numbers:

```markdown
| Your Project | Legacy Estimate | Actual AI Time | Speedup |
|-------------|:-----------:|:----------:|:-------:|
| Feature X    | 80 hrs      | 6 hrs      | 13x     |
| Feature Y    | 40 hrs      | 12 hrs     | 3x      |
```

Your estimates will get more accurate over time as the calibration data grows.

---

## Example

### Input

A spec for a custom mind map control replacing GoJS — interactive canvas with balanced tree layout, expand/collapse, drag, pan/zoom, inline editing, brush color theming, and GoJS JSON import/export.

### Naive 2023 Estimate

~200 hours. Layout engine (40-55hrs), rendering (25-35hrs), interactions (35-50hrs), UI chrome (12-18hrs), polish (30-40hrs).

### Spec Estimator Output

| Metric | Legacy | AI-Single | AI-Parallel (3 agents) |
|--------|:------:|:---------:|:----------------------:|
| **Total Hours** | 200 | 12 | 8 |
| **Human Decision Hours** | 200 | 8 | 8 |
| **Agent Execution Hours** | 0 | 40 | 40 (13 wall-clock) |

**Actual result:** Built and deployed in ~8 hours of human orchestration over 4 weeks.

The Legacy estimate wasn't wrong — it accurately described the work. It just described work that AI agents now do in minutes, not hours.

---

## Contributing

This is an open-source skill. Contributions welcome:

- **Calibration data** — add your real project estimates vs. actuals
- **Friction patterns** — document new friction types you encounter with AI agents
- **Complexity types** — propose new categories as tooling evolves
- **Research prompt templates** — better templates for specific domains (mobile, ML, infra, etc.)

### How to Contribute

1. Fork the repo
2. Add your changes to SKILL.md
3. If adding calibration data, include: project type, legacy estimate, actual AI time, tech stack, and what made it faster or slower than expected
4. Open a PR

---

## License

MIT
