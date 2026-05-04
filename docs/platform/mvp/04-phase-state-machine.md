# Phase State Machine

## Purpose

This document defines the platform phase-state-machine model for the platform MVP.

It covers:
- the generic platform concepts used for phase instances and lifecycle stages
- the concrete standard-ruleset phase cycle currently supported
- visibility, reveal, snapshot, and closing rules that constrain implementation

It does not define a generic workflow engine, scripting system, database schema, API surface, tooling choice, or AI runtime.

---

## Architecture Frame

The platform provides a generic phase instance model.

Rulesets define:
- which phase kinds exist
- transition rules between phase kinds
- which phase kinds are conditional
- what actions are allowed in each phase kind
- what press policy applies in each phase kind and stage
- what reveal policy applies

The platform should support ruleset-defined phase cycles, but it should not hardcode Spring, Fall, or Winter as universal platform concepts. Those are standard-ruleset phase kinds.

The standard ruleset is the only currently specified ruleset.

The platform does not need a generic workflow DSL or scripting engine for the MVP. A small ruleset-defined phase-cycle model is enough.

---

## Core Concepts

### Phase Kind

A **phase kind** answers what kind of phase a phase instance represents.

Phase kinds are ruleset-defined. The platform should not assume that every ruleset has Spring, Fall, Winter, movement, retreat, or adjustment phases.

For the standard ruleset, the supported phase kinds are:
- `spring_movement`
- `spring_retreats`
- `fall_movement`
- `fall_retreats`
- `winter_adjustments`

### Phase Stage

A **phase stage** answers where a phase instance is in its lifecycle.

Phase stage is separate from phase kind. For example, `spring_movement` is a phase kind, while `open_for_input` or `resolved_revealed` is a lifecycle stage of a particular phase instance.

### Phase Instance

A **phase instance** is one occurrence of one phase kind in one game.

Examples:
- Spring 1901 movement
- Spring 1901 retreats
- Fall 1901 movement
- Winter 1901 adjustments

Each phase instance has:
- a game
- a year or equivalent ruleset-defined sequence marker
- a phase kind
- a phase stage
- links to previous and next phase instances when known

### Transition Rule

A **transition rule** determines which phase kind opens after the current phase reaches its resolved/revealed boundary.

Transition rules belong to the ruleset. The platform applies them without treating the standard Diplomacy cycle as universal platform behavior.

### Conditional Phase

A **conditional phase** opens only when the resolved state creates work for that phase.

In the standard ruleset, retreat phases are conditional. A spring or fall retreat phase opens only when at least one unit must retreat or disband after the corresponding movement phase.

### Reveal Boundary

A **reveal boundary** is the point where a phase reaches `resolved_revealed`.

At this boundary:
- adjudication or phase resolution has completed
- finalized submissions for the phase become visible according to reveal policy
- phase results become visible according to visibility rules
- the resolved phase snapshot is taken

---

## Platform-Standard Phase Stages

The platform uses the following phase stages for the MVP.

### `open_for_input`

The phase is open for allowed seat actions.

Examples include movement orders, retreat choices, adjustment choices, and press where the phase's press policy allows it.

### `input_locked`

The phase is no longer accepting seat input.

Submitted orders or actions are fixed for resolution. This stage exists even if the lock happens immediately because no seat has meaningful input.

### `resolving`

The phase is being resolved.

For movement phases, this means adjudication is running or being applied. For retreat and adjustment phases, this means the platform is resolving the submitted retreat or adjustment actions.

### `resolved_revealed`

The phase has resolved and results have been revealed.

Full finalized submissions for the phase become visible at this stage according to the reveal policy. The phase may remain the current active phase while players inspect results, even though no further seat actions are accepted for that phase.

The resolved phase snapshot is taken at this stable boundary.

### `closed`

The phase is closed.

A phase becomes `closed` when the next phase opens. `closed` is distinct from `resolved_revealed`; a phase can be resolved and revealed before it is closed.

---

## Standard-Ruleset Phase Kinds

The standard ruleset currently defines these phase kinds.

### `spring_movement`

Seats with active units may submit spring movement orders.

### `spring_retreats`

Seats with dislodged units from spring movement may submit retreat or disband choices.

This phase is conditional.

### `fall_movement`

Seats with active units may submit fall movement orders.

### `fall_retreats`

Seats with dislodged units from fall movement may submit retreat or disband choices.

This phase is conditional.

### `winter_adjustments`

Seats may submit required build or disband actions when their supply-center count and unit count differ.

Winter adjustments still exist as a phase in the standard cycle even if no seat has a meaningful adjustment action. In that case, the phase may move through input and resolution quickly, but it should still be represented as a phase instance.

---

## Standard-Ruleset Annual Cycle

The standard-ruleset annual cycle is:

1. `spring_movement`
2. `spring_retreats`, if needed
3. `fall_movement`
4. `fall_retreats`, if needed
5. `winter_adjustments`
6. next year `spring_movement`

Transition rules:
- after `spring_movement`, open `spring_retreats` if any retreat or disband requirement exists; otherwise open `fall_movement`
- after `spring_retreats`, open `fall_movement`
- after `fall_movement`, open `fall_retreats` if any retreat or disband requirement exists; otherwise open `winter_adjustments`
- after `fall_retreats`, open `winter_adjustments`
- after `winter_adjustments`, open next year `spring_movement`

Retreat phases are conditional.
Winter adjustment phases are not skipped from the standard cycle solely because no seat has a meaningful adjustment action.

---

## Stage Behavior By Phase Kind

### `spring_movement`

During `open_for_input`:
- seats with active units may draft, revise, and submit movement orders
- press is allowed according to the game press policy

At `resolved_revealed`:
- finalized movement submissions become visible
- adjudication results become visible
- retreat requirements, if any, become visible to affected seats and admin/testing views according to visibility rules

### `spring_retreats`

During `open_for_input`:
- only seats with spring retreat or disband requirements have required retreat actions
- press is allowed according to the game press policy

At `resolved_revealed`:
- finalized retreat/disband submissions become visible
- retreat outcomes become visible
- the resulting board state becomes visible

### `fall_movement`

During `open_for_input`:
- seats with active units may draft, revise, and submit movement orders
- press is allowed according to the game press policy

At `resolved_revealed`:
- finalized movement submissions become visible
- adjudication results become visible
- retreat requirements, if any, become visible to affected seats and admin/testing views according to visibility rules
- fall supply-center occupation results needed for later adjustment resolution are available to the platform

### `fall_retreats`

During `open_for_input`:
- only seats with fall retreat or disband requirements have required retreat actions
- press is allowed according to the game press policy

At `resolved_revealed`:
- finalized retreat/disband submissions become visible
- retreat outcomes become visible
- the resulting board state and supply-center control changes become visible

### `winter_adjustments`

During `open_for_input`:
- seats with required builds or disbands may draft, revise, and submit adjustment actions
- seats with no required adjustment have no meaningful adjustment action
- press is allowed according to the game press policy

At `resolved_revealed`:
- finalized adjustment submissions become visible
- build and disband results become visible
- the resulting board state becomes visible

---

## Visibility And Reveal Rules

Finalized submissions for a phase become visible at `resolved_revealed`.

Before `resolved_revealed`:
- a seat can see its own drafts and effective submissions
- a seat cannot see other seats' private submissions unless an explicit admin/testing view is being used
- admin/testing views may expose broader information only as explicit admin/testing surfaces

At `resolved_revealed`:
- finalized submissions for the phase become visible according to the phase reveal policy
- phase results become visible according to seat visibility rules
- adjudication explanations and resolution entries may become visible according to seat/admin read-model policy

Messages remain first-class communication objects and are visible according to channel membership and phase linkage. Message visibility is not replaced by phase reveal.

---

## Snapshot And History Boundary

Resolved phase snapshots are taken at stable `resolved_revealed` boundaries.

Snapshots are not taken at every stage transition. In particular, the platform should not create phase snapshots merely because a phase opens for input, locks input, or enters resolving.

Event history may record stage transitions, submissions, adjudication runs, message sends, reveals, and phase openings. Event history supports auditability and replay, but it does not replace current canonical state or resolved phase snapshots.

---

## Closing Rule

A phase becomes `closed` when the next phase opens.

This means:
- a phase can be `resolved_revealed` and still be the current active phase
- opening the next phase closes the previous phase
- `closed` should not be collapsed into `resolved_revealed`

---

## Future Flexibility

The platform phase model is generic enough to support future rulesets with:
- different phase vocabularies
- different transition rules
- different conditional phase rules
- different press policies
- different reveal policies
- more granular phase kinds

This flexibility should come from ruleset-defined phase-cycle metadata and platform boundaries, not from a broad generic workflow engine.

The platform MVP should implement the standard-ruleset cycle directly and cleanly while preserving the ruleset boundary for later extension.
