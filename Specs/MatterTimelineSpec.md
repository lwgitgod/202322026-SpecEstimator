# Custom Matter Timeline Control - Product-Level Functional Specification

Baseline Replacement for CaseFleet Visual Timeline

## 1. Purpose

1. We are replacing the CaseFleet visual timeline component with our own fully owned implementation.
2. The goal is not to invent a new interaction model. The goal is to match and slightly modernize the user experience of a professional legal case timeline, similar to the visual timeline view in CaseFleet (casefleet.com/features/timelines).
3. The component must feel complete, stable, polished, and production ready. It should behave exactly how a user expects a professional litigation timeline tool to behave.
4. This document describes what the control must do. It intentionally avoids technical architecture and class structure.

---

## 2. What We Are Building

We are building an interactive, client-side matter timeline control that:

* Displays chronological events along a horizontal time axis
* Organizes events into swimlanes by party or category
* Supports collapsible litigation phases
* Supports pan and zoom across the time axis
* Supports inline date and label editing
* Supports event category color theming
* Can import and export a flat JSON event array

This replaces CaseFleet's visual timeline entirely. It must not depend on CaseFleet or any licensed timeline library internally.

---

## 3. Core Behavior

### Time Axis and Chronological Layout

* The horizontal axis represents time.
* The axis is continuous and scrollable, not paginated.
* Time scale adapts to the data range:

  * Days if the matter spans weeks
  * Weeks if the matter spans months
  * Months if the matter spans years
* Tick marks and date labels appear at regular intervals along the axis.
* A "today" marker is always visible when the current date falls within the visible range.

The time axis is not decorative. It is the primary structural anchor of the entire control.

---

### Swimlane Layout

Events are grouped into horizontal swimlanes.

Each swimlane represents a party, entity, or category relevant to the matter. The system:

* Stacks swimlanes vertically, each with a labeled header on the left
* Places events within their assigned lane at the correct horizontal position based on date
* Prevents event overlap within a lane by stacking or offsetting when dates collide
* Maintains consistent lane height, expanding only when stacking requires it
* Supports a shared "Court / Tribunal" lane that is always visible at the top

Swimlane order is determined by the data. The first lane in the import array renders at the top.

---

### Litigation Phases

Events can be grouped into litigation phases.

Phases are collapsible horizontal bands that span a date range and contain one or more events. Built-in phase types include:

* Pleadings
* Discovery
* Motions
* Pre-Trial
* Trial
* Post-Trial / Appeal

Phases are rendered as subtle shaded regions behind the swimlanes, spanning their start and end dates.

* Clicking a phase header collapses it, hiding all contained events
* Collapsed phases show a summary chip with the event count
* Phases can overlap in time
* Events belong to at most one phase
* Events with no phase assignment are always visible

Phase collapsing is for focus and readability, not data filtering.

---

## 4. Event Types and Visual Identity

There are four event types:

* Filing
* Deadline
* Court Date
* General Event

### Visual Styling

Events are compact, rounded cards with subtle shadows and clean borders.

* Filings use a document icon and a blue-toned palette
* Deadlines use a clock icon and a red-toned palette
* Court Dates use a gavel icon and a purple-toned palette
* General Events use a neutral gray palette

### Category Color Support

Categories are color themes assigned per event type or per custom tag.

Each category:

* Defines the event card border and accent color
* Applies a coordinated light background fill
* Renders consistently across swimlanes and phases
* Displays in the legend

If custom tags are used, colors are assigned from a predefined palette and cycle if tags exceed the palette size.

The Court / Tribunal swimlane always uses its own distinct visual identity.

---

## 5. Expand and Collapse

Litigation phases can be expanded and collapsed.

* Clicking a phase header toggles it
* Toggle indicators clearly show expanded or collapsed state
* Collapsing hides all events within the phase date range that are assigned to that phase
* Expand All and Collapse All are supported globally
* The timeline loads fully expanded by default
* The Court / Tribunal lane cannot be collapsed

Expand All and Collapse All perform full recalculations of vertical layout.

During layout updates:

* Controls are temporarily disabled
* A visual "updating" indicator appears

---

## 6. Event Interaction

Events support hover and click interaction.

* Hovering an event shows a tooltip with full details: date, title, description, party, phase, and category
* Clicking an event selects it:

  * Highlights the event card
  * Displays a detail panel on the right side of the viewport
  * Draws a vertical time marker across all swimlanes at the event date
* Clicking the canvas background deselects

Selection is single-select only. There is no multi-select.

---

## 7. Pan and Zoom

The canvas supports:

* Click-and-drag horizontal panning along the time axis
* Scroll wheel zoom on the time axis
* Vertical scrolling for swimlanes that exceed the viewport
* Zoom range from full-matter overview to single-day granularity
* Default zoom fits the entire matter date range into view

A Fit button:

* Fits the entire matter timeline into view
* Adds padding on both ends
* Does not zoom past one-month-per-screen-width granularity
* Animates smoothly

A Jump to Today button:

* Centers the viewport on today's date
* Animates smoothly
* Is disabled if the matter has no events near the current date

Fit-to-view runs on initial load and on window resize.

---

## 8. Inline Editing

Users can edit event details directly in the timeline.

* Double-click on an event date activates a date picker inline
* Double-click on an event label activates text editing
* Enter or blur saves
* Escape cancels
* Changes persist for the session only
* Editing does not trigger full layout recalculation unless the date changes
* Date edits reposition the event on the time axis immediately
* While editing, phase collapse actions are suppressed

Each event supports:

* Title (required)
* Date (required)
* End date (optional, for events with duration)
* Description (optional)

Text truncates with ellipsis within the event card. Full text is visible in the tooltip and detail panel.

---

## 9. Connectors and Time Markers

Relationships between events are shown as optional connector lines.

* Connectors are subtle dashed lines between related events
* No arrowheads
* Lines route horizontally along the time axis then vertically between lanes
* Connectors are defined in the data, not inferred

Time markers are vertical lines drawn across all swimlanes:

* Selected event marker: solid, accented
* Today marker: dashed, muted
* Deadline markers: dotted, red-toned, drawn only when deadlines are visible

Markers do not interfere with event interaction.

---

## 10. Control Panel and UI

The interface includes:

Top Center Panel:

* Matter title
* Expand All
* Collapse All
* Fit
* Jump to Today
* Import
* Export
* Updating indicator

Left Side:

* Swimlane headers with party/category names
* Phase headers with collapse toggles

Bottom Left:

* Legend for event types and category colors

Bottom Right:

* Minimap showing the full timeline with a viewport indicator

Top Left:

* Zoom controls (zoom in, zoom out, zoom level display)

Right Side (conditional):

* Event detail panel, visible when an event is selected

The canvas fills the viewport and includes a subtle alternating-row background for swimlane readability.

---

## 11. Import and Export

The system must support a flat JSON event array format.

### Export

Exports a flat array where each object includes:

* id
* title
* date (ISO 8601)
* endDate (ISO 8601, optional)
* description
* party
* phase
* category
* type (filing | deadline | courtDate | event)
* relatedEvents (array of ids, optional)

Downloads as JSON.

### Import

Accepts a JSON file containing an array of event objects.

* Reconstructs swimlanes from unique party values
* Determines phase groupings from phase values
* Assigns event types from type field
* Assigns category colors from category values
* Ignores viewport state and recalculates layout
* Expands all phases on import

Round-trip must preserve:

* All event data fields
* Swimlane assignments
* Phase assignments
* Category assignments
* Connector relationships

It does not preserve:

* Viewport position
* Zoom level
* Selection state
* Swimlane ordering (re-derived from data on each import)

---

## 12. What This Baseline Does Not Include

Explicitly out of scope:

* Keyboard navigation
* Full-text search across events
* Undo or redo
* Multi-select
* Image or PDF export
* Live collaboration
* Server persistence
* Event creation or deletion from UI
* Drag-to-reschedule
* Document attachment or linking
* Dark mode
* Vertical timeline orientation
* Witness or evidence linking
* AI-assisted timeline generation

---

## 13. What Success Looks Like

If someone interacts with this control and has previously used CaseFleet's visual timeline or a comparable litigation timeline tool, they should immediately recognize the behavior.

It should:

* Feel stable
* Feel predictable
* Handle matters with hundreds of events without visual breakdown
* Support swimlane separation cleanly
* Support phase expand and collapse without layout degradation
* Round-trip data without loss
* Never visually degrade after repeated zoom, pan, or collapse interactions
