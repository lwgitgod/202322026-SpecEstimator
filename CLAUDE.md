# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is **Spec Estimator** — a Claude Code skill (not a traditional software project). It's a markdown-based tool that takes technical specifications and produces effort estimates calibrated for AI-assisted development (2026). There is no runtime, no dependencies, no build step, and no `node_modules`.

The core deliverable is `SKILL.md` — a structured prompt that Claude reads to perform estimation. The skill lives in `.claude/skills/SKILL.md` and is also mirrored at the repo root.

## Repository Structure

```
SKILL.md                          # The skill definition (core artifact)
.claude/skills/SKILL.md           # Installed skill location (same content)
GoJS_replacementSpec.md           # Example spec used as calibration reference
.claude/agents/                   # Custom agent definitions for team workflows
.claude/commands/                 # Slash commands (CommandOne through CommandFour, 2026Estimate)
.claude/tasks/                    # Task definitions for agent orchestration
```

## How the Skill Works

1. User provides a spec (any format: markdown, doc, PDF, plain text)
2. Skill decomposes the spec into functional areas
3. Each area is classified by complexity type (Boilerplate, Library Integration, Novel SDK, Algorithmic, Architectural, Creative/UX, Integration Glue, Unknown/Research)
4. A research prompt is generated for unknowns
5. A three-column estimate is produced: Legacy (2023 human-solo), AI-Single, AI-Parallel

The estimation model uses empirically-derived friction allowances (35% wrong-approach rate, 25% SDK errors, 20% misunderstandings, 30% integration bugs) applied multiplicatively.

## Agent Team

This repo defines 8 custom agents in `.claude/agents/`. Agent teams are enabled via `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in `.claude/settings.json`.

| Agent | Model | Purpose |
|-------|-------|---------|
| `TECH-full-stack-architect` | opus | End-to-end feature implementation (Python/FastAPI + Next.js/React) |
| `BA-tech-spec-writer` | opus | Writes technical specs from requirements with mandatory complexity justification |
| `BA-tech-lead-skeptic` | opus | Pre/post-implementation review — challenges unjustified complexity |
| `backlog-analyst` | opus | Converts specs/PRDs into precise backlog items, eliminates AI-generated fluff |
| `backend-architect` | opus | Database-first backend design (PostgreSQL stored procs, FastAPI) |
| `senior-coder` | sonnet | Feature implementation with TDD approach |
| `frontend-architect` | sonnet | Next.js 14 App Router, bulletproof-react pattern enforcement |
| `spec-wireframe-builder` | sonnet | Product specs + .DRAWIO wireframes, desktop-only (no mobile) |

## Slash Commands

- `/CommandOne` — Implements task1.md using TECH-full-stack-architect
- `/CommandTwo` — Implements Track B Phase 6 tickets using TECH-full-stack-architect
- `/CommandThree` — Generates CoreAPIService.py using BA-tech-spec-writer
- `/CommandFour` — Implements hybrid factual extraction using TECH-full-stack-architect
- `/2026Estimate` — Runs the spec estimator skill

## Key Technical Patterns (from agents)

These patterns are enforced across the agent team for the parent project (Legawrite AI):

**Backend**: Python 3.10+ / FastAPI / PostgreSQL with stored procedures. Three-layer architecture: routers → services → repositories. Every table has `id`, `created_at`, `updated_at`. Stored procedures get `COMMENT`. Use Pydantic for validation.

**Frontend**: Next.js 14 App Router / React / TypeScript / Material-UI v6 (`mui_v2`). Mandatory bulletproof-react 4-part API pattern (pure function → query options → TypeScript config → custom hook). Mandatory 3-layer page pattern (`page.tsx` → `PageContent` → `Content`). Path aliases: `@/*` → `src/*`, `@/UI2/*` → `src/ui/mui_v2/*`. Never use `any` in TypeScript.

**Non-negotiable rules**: Never connect to remote Azure databases. Never run migration scripts directly. Never use `any` in TypeScript. Always use `APIContext.isReady` check in React Query hooks.

## Modifying the Skill

When editing `SKILL.md`, keep both copies in sync (root `SKILL.md` and `.claude/skills/SKILL.md`). Changes to the estimation model (friction rates, calibration benchmarks, complexity types) should be grounded in empirical data. The calibration table at the bottom of SKILL.md should be updated with real project data as it becomes available.
