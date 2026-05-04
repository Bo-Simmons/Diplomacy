# Architecture Overview

## Purpose

This document defines the top-level architecture frame for the repository.

The repository is intended to contain two closely related but separate long-term systems:

1. the **Diplomacy platform**
2. the future **AI-player subsystem**

These systems belong in one repository because they share game concepts, visibility rules, history models, and seat-facing contracts. They must remain separate subsystems because the platform needs to be useful before AI exists, and future AI players should plug into the platform rather than become part of the platform core.

The current active build target is the **platform MVP**.

---

## Current Implementation Focus

Current work should focus on the platform MVP only.

The platform MVP is a local-first, server-authoritative Diplomacy platform that can run fully on one developer machine. It should support standard-rules play, multiple seat perspectives for local testing, universal press, direct two-seat press, structured diplomacy history, and a clean adjudication boundary.

The AI-player subsystem is deferred. Current platform design should preserve clean boundaries for future AI integration, but should not implement AI players, model/provider abstractions, AI memory, negotiation engines, or prompt systems unless explicitly requested.

---

## Top-Level Subsystems

### Platform

The platform is the active subsystem for the MVP.

It owns:
- game/session orchestration
- ruleset and map definitions
- runtime game state
- order collection and submission
- adjudication integration boundaries
- press, messaging, and diplomacy history
- seat-scoped and admin/testing read models
- local-first persistence and history

The platform must remain usable with only human-operated seats.

### AI

The AI subsystem is a future subsystem.

It is expected to operate as a seat occupant through platform-defined interfaces. It should not replace the platform game loop, adjudicator, messaging layer, visibility model, or history model.

Detailed AI architecture is intentionally deferred until the platform has stable enough contracts to design against.

### Shared

Shared concepts are the durable contracts and terminology used by both platform and future AI work.

Shared concerns include:
- variant and map data concepts
- visibility and permissions
- event history and replay concepts
- canonical terminology
- seat-facing state and history contracts

Shared docs should describe concepts and contracts, not secretly implement either subsystem.

---

## Platform Architecture

The platform should be organized around explicit domain boundaries rather than implementation convenience.

### Game And Session Orchestration

The platform owns game creation, game status, phase advancement, seat assignment, and local admin/testing controls.

The game/session layer coordinates the current phase, active seats, operational configuration, and phase transitions. It should not contain adjudication rules, AI reasoning, or UI-specific read shaping.

### Ruleset And Map Definitions

Rulesets and maps are separate concepts.

A ruleset defines gameplay semantics, such as standard Diplomacy phase and order concepts. A map exists under a ruleset and defines topology, powers, starting units, starting supply center control, province metadata, board locations, and rendering metadata.

The MVP only needs standard Diplomacy rules, but map and setup data should remain data-driven from the beginning.

### Runtime Game State

The platform maintains the current operational truth of the game as current canonical state.

Current canonical state is distinct from:
- resolved phase snapshots
- append-only event history

The platform should not require full event replay to answer the present game situation.

### Orders And Adjudication Boundary

Orders should be structured platform objects, not plain text blobs.

The adjudicator should remain a clean boundary. Conceptually, it accepts variant/setup data, current game state, and submitted orders. It returns resolved state, adjudication results, retreat requirements, adjustment requirements, and phase outputs.

Adjudication must not own messaging, diplomacy artifacts, UI rendering, AI planning, or admin/testing access.

### Press, Messaging, And Diplomacy History

Press and diplomacy history are first-class platform concerns.

The MVP communication model includes:
- universal press visible to all seats
- direct press visible only to exactly two participating seats

Messages are phase-linked first-class entities. Message sending may also appear in event history, but messages must not be reduced to generic event rows.

Structured diplomacy records, including commitments and possible future labels such as Agreement, Pact, or Treaty, are communication-layer artifacts. Their exact taxonomy remains provisional. They are socially meaningful and queryable, but they are not mechanically binding and must not be enforced by the adjudicator.

### Read Models

Read models are separate from normalized core domain entities.

The platform should assemble seat-scoped read models that expose only what a seat is allowed to see. Admin/testing read models may expose broader information for local development and debugging, but they must remain explicit admin/testing surfaces rather than gameplay shortcuts.

### Persistence And History

The platform should maintain three distinct history/state concepts:

- **current canonical state**: live operational truth for the current game situation
- **resolved phase snapshots**: frozen checkpoints at stable resolved/revealed phase boundaries
- **event history**: append-only chronology for auditability, replay, debugging, and future AI-facing history use

Resolved snapshots and event history support understanding the past. Current canonical state supports operating the game now.

---

## Locked Platform Modeling Decisions

The following decisions are established for the platform MVP:

- `Seat` is the primary gameplay actor boundary.
- `Ruleset` and `Map` are separate, with maps under rulesets.
- `Province` and `BoardLocation` are separate.
- Board locations are explicit.
- Units occupy board locations, not bare provinces.
- Land and fleet adjacency are separate stored graphs.
- Convoy reachability is derived, not stored as a third explicit graph.
- Inaccessible named regions are map features, not provinces.
- `Phase` is explicit and typed.
- Phase type and phase stage are separate.
- `resolved_revealed` and `closed` are distinct phase stages.
- Current canonical state, resolved phase snapshots, and event history are distinct.
- Messages are first-class communication objects.
- Commitments are made through messages and linked back to specific messages.
- Read models are separate from core domain entities.

---

## Boundaries That Must Remain Clean

The following boundaries are architectural constraints:

- Adjudication is separate from messaging and diplomacy history.
- Adjudication is separate from UI rendering and read-model shaping.
- The platform is separate from the future AI runtime.
- Admin/testing access is separate from gameplay rules.
- Seat visibility is enforced through explicit visibility/read-model boundaries.
- Variant/map data is separate from rules logic.
- Event history does not replace first-class domain entities.
- Communication-layer diplomacy artifacts are not mechanically enforced by the judge.

These boundaries should guide implementation choices and future documentation.

---

## Admin And Testing Architecture

The MVP should be fully testable by one human operator on one machine.

The local admin/testing workflow should allow a single operator to create games, inspect game progress, advance phases, and open or simulate multiple seat perspectives. This workflow is for local development and testing, not a gameplay rule.

Even in local-only mode, seat-scoped visibility must remain intact. A seat view should only show what that seat is allowed to see. Broader visibility belongs in explicit admin/testing views.

This distinction matters because future AI players will depend on the same seat-scoped boundary that human players use.

---

## Future AI Boundary

The future AI subsystem should sit on top of the platform.

An AI player is expected to act as a seat occupant through platform-defined interfaces. It should consume seat-visible state, seat-visible messages, legal actions, history, and diplomacy records according to the same visibility rules that apply to any other seat occupant.

The AI subsystem should not be implemented as special-case game logic inside the platform. It should not bypass adjudication, visibility, messaging, or phase lifecycle boundaries.

Detailed AI-player design belongs in future AI docs after the platform contracts are stable enough to design against.

---

## Relationship To Other Docs

Read this document as the repository-level architecture frame.

Related docs:
- Repository roadmap: `docs/overview/repository-roadmap.md`
- Platform MVP domain model: `docs/platform/mvp/03-domain-model.md`
- Future platform MVP docs: `docs/platform/mvp/`
- Future shared docs: `docs/shared/`
- Future AI overview docs: `docs/ai/overview/`

This document should stay architecture-level. Detailed domain entities belong in the domain model. Implementation plans, API surfaces, database schema, tooling decisions, and detailed AI design belong in their own focused docs.
