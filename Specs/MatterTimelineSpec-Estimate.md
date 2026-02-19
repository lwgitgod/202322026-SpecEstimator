# Effort Estimate: Custom Matter Timeline Control

Baseline Replacement for CaseFleet Visual Timeline

---

## Spec Quality Assessment

| Dimension | Score | Notes |
|-----------|:-----:|-------|
| Behavioral Clarity | 5/5 | Interactions described precisely — hover, click, double-click, pan, zoom, collapse all have explicit behaviors defined |
| Scope Boundaries | 5/5 | Exceptional. Section 12 explicitly excludes 14 features. Crystal clear what's out. |
| Data Contracts | 5/5 | Import/export JSON schema fully defined with field names, types, and round-trip guarantees |
| Success Criteria | 4/5 | Section 13 defines qualitative success well; lacks quantitative perf benchmarks (e.g., "100 events at 60fps") |
| Technical Specificity | 3/5 | Intentionally stack-agnostic ("avoids technical architecture"). No framework named. This is a design choice, not a gap, but it means the estimator must assume React. |
| Ambiguity Level | 4/5 | Very few open questions. Minor: connector routing algorithm details, minimap interaction behavior, exact zoom step sizes |
| **Average** | **4.3/5** | **Spec Quality Multiplier: 0.8x** |

**Assessment:** This is a good-to-exceptional spec. Behavioral clarity and scope boundaries are best-in-class. The intentional omission of tech stack is the only meaningful gap, and it's a deliberate design choice. Applying 0.8x multiplier.

---

## Complexity Mix Profile

| Complexity Type | Functional Areas | % of Legacy Effort | Factor | Weighted Contribution |
|---|---|:---:|:---:|:---:|
| Boilerplate | Control panel UI, legend, detail panel, import/export, event cards, toolbar buttons | 35% | 12x | 4.20 |
| Algorithmic | Time axis scaling, swimlane layout engine, collision avoidance, phase region rendering, zoom/pan coordinate math | 25% | 6x | 1.50 |
| Library Integration | Canvas/SVG rendering, date picker, zoom/pan gesture handling, JSON file I/O | 20% | 8x | 1.60 |
| Creative/UX | Event card visual design, color theming, animations, minimap, tooltip polish | 12% | 4x | 0.48 |
| Integration Glue | Connecting layout engine to renderer, edit-mode ↔ layout suppression, connector routing, zoom ↔ event repositioning | 8% | 4x | 0.32 |
| **Blended Factor** | | **100%** | | **8.1x** |

**Profile match:** Single-domain (all frontend, client-side only), mixed complexity. Falls squarely in the "Medium single-domain, mixed complexity" range (5-15 hrs AI-assisted). The heavy boilerplate and tight spec push it toward the faster end.

---

## Summary

| Metric | Legacy (2023) | AI-Single | AI-Parallel |
|--------|:------------:|:---------:|:-----------:|
| **Total Hours** | 75 | 12 | 7 |
| **Calendar Days** (6hr productive/day) | 12.5 | 2.0 | 1.2 |
| **Human Decision Hours** | — | 5 | 4 |
| **Agent Execution Hours** | — | 7 | 3 |

**Formula applied:**
- AI-Single = 75 ÷ 8.1 × 0.8 (spec quality) × 1.25 (friction) = **9.3 hrs base → ~12 hrs with review buffer**
- AI-Parallel = 12 × 0.6 (3 parallel streams) = **~7 hrs**

---

## Decomposition

### 1. Time Axis and Coordinate System
- **What:** Continuous horizontal time axis with adaptive scale (days/weeks/months), tick marks, date labels, today marker. Core coordinate math: date↔pixel mapping at any zoom level.
- **Complexity Type:** Algorithmic
- **Legacy:** 12 hrs — Custom scale calculation, adaptive tick placement, smooth zoom transforms
- **AI-Single:** 2.0 hrs — Well-understood math (linear interpolation + d3-scale patterns), spec is precise on behavior. Factor 6x × 0.8 spec quality × 1.15 friction = 2.0
- **AI-Parallel:** 1.5 hrs — Can build independently, but other areas depend on this as foundation
- **Confidence:** High — This is a solved problem with extensive prior art

### 2. Swimlane Layout Engine
- **What:** Vertical stacking of labeled lanes, event placement by date, collision avoidance when events overlap, dynamic lane height expansion. Court/Tribunal lane pinned at top.
- **Complexity Type:** Algorithmic
- **Legacy:** 10 hrs — Collision detection and stacking logic, dynamic height recalculation
- **AI-Single:** 1.8 hrs — Standard layout algorithm, spec clearly defines stacking behavior. Factor 6x × 0.8 × 1.15 = 1.8
- **AI-Parallel:** 1.2 hrs — Depends on time axis coordinate system
- **Confidence:** High — Stacking/collision is well-documented pattern

### 3. Event Cards and Visual Identity
- **What:** Four event types (Filing, Deadline, Court Date, General) with distinct icons, color palettes, rounded card styling. Category color theming with palette cycling. Text truncation with ellipsis.
- **Complexity Type:** Boilerplate + Creative/UX
- **Legacy:** 8 hrs — Styling four variants, icon integration, color system
- **AI-Single:** 0.8 hrs — Straightforward component work, spec defines exact visual rules. Factor 10x × 0.8 × 1.1 = 0.8
- **AI-Parallel:** 0.5 hrs — Fully independent of layout engine
- **Confidence:** High — Pure UI component work

### 4. Litigation Phases (Collapse/Expand)
- **What:** Shaded background regions spanning date ranges, collapsible with header click, summary chips when collapsed, expand/collapse all, layout recalculation with updating indicator. Court lane immune to collapse.
- **Complexity Type:** Algorithmic + Boilerplate
- **Legacy:** 10 hrs — Phase region rendering behind swimlanes, expand/collapse state management, layout recalc triggers
- **AI-Single:** 1.5 hrs — Moderately complex state management but spec is very precise on behavior. Factor 7x × 0.8 × 1.2 = 1.5
- **AI-Parallel:** 1.0 hrs — Partially parallel, depends on layout engine
- **Confidence:** Medium-High — The interaction between phase collapse and layout recalc is the trickiest part

### 5. Pan and Zoom
- **What:** Click-drag horizontal pan, scroll-wheel zoom, vertical scroll for overflow, zoom range constraints, Fit button (animate to full range), Jump to Today button. Fit on load and resize.
- **Complexity Type:** Library Integration + Algorithmic
- **Legacy:** 8 hrs — Gesture handling, zoom math, animation, viewport management
- **AI-Single:** 1.2 hrs — Standard pattern with d3-zoom or similar. Spec defines exact constraints. Factor 7x × 0.8 × 1.15 = 1.2
- **AI-Parallel:** 0.8 hrs — Can build in parallel with layout engine once coordinate system exists
- **Confidence:** High — d3-zoom or similar handles 80% of this

### 6. Event Interaction (Hover, Select, Detail Panel)
- **What:** Tooltip on hover (full details), single-select on click with highlight + detail panel + vertical time marker, deselect on background click.
- **Complexity Type:** Boilerplate + Creative/UX
- **Legacy:** 6 hrs — Tooltip positioning, detail panel layout, selection state management
- **AI-Single:** 0.7 hrs — Standard interaction patterns, spec is explicit. Factor 10x × 0.8 × 1.1 = 0.7
- **AI-Parallel:** 0.4 hrs — Fully independent
- **Confidence:** High — Standard UI interaction

### 7. Inline Editing
- **What:** Double-click to edit date (date picker) or label (text input). Enter/blur saves, Escape cancels. Date edits reposition event. Suppress phase collapse during edit.
- **Complexity Type:** Library Integration + Integration Glue
- **Legacy:** 6 hrs — Date picker integration, inline text editing, edit-mode state management
- **AI-Single:** 1.0 hrs — Date picker is library work, text editing is standard. The edit-mode suppression adds friction. Factor 6x × 0.8 × 1.25 = 1.0
- **AI-Parallel:** 0.7 hrs — Partially parallel
- **Confidence:** Medium-High — Edit-mode ↔ layout interaction needs careful testing

### 8. Connectors and Time Markers
- **What:** Dashed connector lines between related events, horizontal-then-vertical routing, no arrowheads. Three marker types: selected (solid), today (dashed), deadline (dotted red).
- **Complexity Type:** Algorithmic + Boilerplate
- **Legacy:** 5 hrs — Line routing algorithm, marker rendering, z-order management
- **AI-Single:** 0.8 hrs — Routing is simple (horizontal + vertical), markers are straightforward. Factor 7x × 0.8 × 1.15 = 0.8
- **AI-Parallel:** 0.5 hrs — Fully independent once layout positions are known
- **Confidence:** High — Simple routing, no complex path-finding

### 9. Control Panel and Toolbar
- **What:** Top-center toolbar (title, 7 buttons, updating indicator), zoom controls (top-left), legend (bottom-left), minimap (bottom-right).
- **Complexity Type:** Boilerplate + Creative/UX (minimap)
- **Legacy:** 6 hrs — Layout, button wiring, minimap viewport indicator
- **AI-Single:** 0.8 hrs — Mostly boilerplate. Minimap is the only non-trivial piece. Factor 9x × 0.8 × 1.1 = 0.8
- **AI-Parallel:** 0.4 hrs — Fully independent
- **Confidence:** High — Standard toolbar + minimap is a known pattern

### 10. Import/Export
- **What:** JSON file export (flat array with defined schema), JSON file import (reconstruct swimlanes, phases, categories from data), round-trip fidelity guarantees.
- **Complexity Type:** Boilerplate
- **Legacy:** 4 hrs — File I/O, JSON parsing/serialization, data reconstruction
- **AI-Single:** 0.4 hrs — Trivial for AI. Schema is fully defined. Factor 12x × 0.8 × 1.05 = 0.4
- **AI-Parallel:** 0.2 hrs — Fully independent
- **Confidence:** High — Pure data transformation

---

## Risk Map

| Risk | Impact on Estimate | Likelihood | Mitigation |
|------|-------------------|------------|------------|
| Library choice mismatch (chosen rendering lib doesn't support a needed feature) | +3-5 hrs | Low | Research phase; SVAR Gantt or D3+SVG both cover requirements |
| Phase collapse ↔ layout engine interaction bugs | +1-2 hrs | Medium | Integration test the collapse/expand cycle early |
| Minimap viewport sync at edge zoom levels | +0.5-1 hr | Medium | Constrain minimap to simple proportional rectangle |
| Performance with 300+ events at deep zoom | +1-2 hrs | Low | SVG handles this range well; virtualize if needed |
| Date picker inline positioning in zoomed/panned state | +0.5-1 hr | Medium | Use absolute positioning relative to viewport, not canvas |
| Tech stack not specified — wrong framework assumption | +2-4 hrs | Low | Assume React (most likely given the parent project context) |

**Total risk buffer:** 2-4 hrs (already partially included in friction allowances)

---

## Recommended Agent Strategy

### Parallel Workstreams

**Stream A — Core Engine (sequential foundation)**
Time axis coordinate system → Swimlane layout engine → Phase regions → Pan/zoom integration

**Stream B — UI Components (fully parallel)**
Event cards (4 types) → Tooltips → Detail panel → Legend → Toolbar → Minimap

**Stream C — Data & Interaction (parallel after coordinate system)**
Import/export → Inline editing → Connectors → Time markers → Selection state

### Suggested Execution Plan

1. **Hour 0-1:** Agent 1 builds the time axis coordinate system and swimlane layout engine (foundation). Human reviews the coordinate math.
2. **Hour 0-1:** Agent 2 builds all event card components, color theming, and toolbar UI (fully independent).
3. **Hour 0-1:** Agent 3 builds import/export and the data model layer (fully independent).
4. **Hour 1-3:** Agent 1 adds phase regions, pan/zoom, and collapse/expand. Agent 2 builds detail panel, tooltips, legend, minimap. Agent 3 builds inline editing, connectors, and time markers.
5. **Hour 3-5:** Integration pass — wire all three streams together. Human reviews interaction between edit mode and layout, collapse and connector visibility, minimap sync.
6. **Hour 5-7:** Polish pass — animations, edge cases, final visual tuning. Human validates against spec success criteria.

**Checkpoints:** Human review at hours 1, 3, and 5.

---

## Research Gaps

### Low-confidence areas requiring research if precision matters:

1. **Rendering library choice** — The spec doesn't name a framework. If building with React (assumed), the choice between SVAR Gantt (pre-built swimlane support), D3.js + custom SVG, or raw React + SVG significantly affects the boilerplate ratio. SVAR Gantt could shift 30% of algorithmic work to library integration (faster). D3 gives more control but more code.

2. **Minimap implementation** — Spec says "minimap showing the full timeline with a viewport indicator." The exact interaction model (can users drag the viewport rectangle? click to jump?) is unspecified. Assumption: view-only proportional indicator.

3. **Connector routing at scale** — With 300+ events and many connectors, naive horizontal-then-vertical routing may produce overlapping lines. The spec says "subtle dashed lines" which suggests visual density isn't a primary concern, but this could need refinement.

### Research Prompt (for deep research if desired)

```markdown
# Estimation Research: Custom Matter Timeline Control

## Context
Building a React-based interactive horizontal timeline for litigation case management.
Displays events on a time axis with swimlanes, collapsible phases, pan/zoom, inline editing,
connectors, and JSON import/export. Single-domain (frontend only), 100-500 events.

## Research Questions

### Libraries & SDKs
- [ ] What is the current state of SVAR React Gantt — does it support custom event card rendering, phase collapse, and connector lines?
- [ ] Can d3-zoom integrate cleanly with React 18/19 without ref management issues?
- [ ] What is the best React-compatible inline date picker that works in a canvas/SVG overlay context?

### Architecture
- [ ] For 100-500 SVG event cards with pan/zoom: is DOM-based SVG sufficient or is virtualization needed?
- [ ] What is the standard approach for minimap ↔ main viewport synchronization in React timeline components?

### Benchmarks
- [ ] What is the typical iteration count for custom timeline controls when using AI coding agents?
- [ ] Are there open-source litigation timeline implementations that can serve as reference architecture?
```

---

## Bottom Line

| | Legacy (2023) | AI-Single | AI-Parallel |
|---|:---:|:---:|:---:|
| **Hours** | **75** | **12** | **7** |
| **Days** | **12.5** | **2** | **1.2** |

This is a **medium single-domain project** with an exceptionally tight spec. The complexity is concentrated in the layout engine and phase interaction — everything else is high-confidence boilerplate or well-known library integration. The explicit exclusion of 14 features (keyboard nav, undo/redo, multi-select, persistence, drag-to-reschedule, etc.) keeps scope firmly bounded.

**Confidence:** Medium-High overall. The layout engine and phase collapse interaction are the main uncertainty drivers. Everything else is straightforward.
