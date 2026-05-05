# Repository Roadmap

## Purpose

This repository is intended to house the long-term architecture and implementation for two related but distinct systems:

1. the **Diplomacy platform**
2. the **AI player system**

Both belong in the same repository, but they are separate subsystems and should remain cleanly decoupled.

The current active focus is the **platform MVP**.

---

## Strategic framing

This project does **not** begin by building AI players.

It begins by building a platform that is strong enough to:
- run standard Diplomacy locally
- support full play and testing by one person across multiple seat perspectives
- expose clean interfaces and durable data models that future AI players can plug into

The platform comes first because the AI system depends on it.

---

## Long-term repository shape

The repository is expected to grow around three major documentation and architecture areas:

- **Platform**
  - the game platform itself
  - adjudication boundary
  - messaging
  - local testing ergonomics
  - UI and API surfaces

- **AI**
  - future AI player system
  - model/provider abstraction
  - per-seat context and memory
  - negotiation and planning loops
  - evaluation and runtime behavior

- **Shared**
  - concepts and schemas used by both platform and AI
  - ruleset/map data model
  - visibility model
  - event log and replay model
  - terminology

Planned documentation structure:

    docs/
    ├── overview/
    ├── platform/
    │   ├── mvp/
    │   └── future/
    ├── ai/
    │   ├── overview/
    │   └── future/
    ├── shared/
    ├── decisions/
    └── prompts/

---

## Current active phase

The repository is currently in the **foundation and design phase**.

The immediate work is:
- repository-level instructions
- roadmap and architecture docs
- MVP platform design docs
- Codex-ready repository guidance
- Codex-ready task briefs

This phase exists to make later implementation deliberate rather than improvisational.

---

## Roadmap phases

## Phase 0 — Repository Foundation

### Purpose
Make the repository legible to both humans and coding agents before serious implementation work begins.

### Goals
- establish durable project instructions
- define the long-term subsystem split
- document the active roadmap
- create the initial planning document set
- make the repo ready for Codex-guided work

### Expected outputs
- `AGENTS.md`
- `docs/overview/repository-roadmap.md`
- `docs/overview/architecture-overview.md`
- `docs/overview/terminology.md`
- initial architecture decision records under `docs/decisions/`

### Notes
This is the current phase.

---

## Phase 1 — Platform MVP-A: Playable Local Platform

### Purpose
Create a standard-rules Diplomacy platform that is playable locally by one person for testing across multiple seats.

### Goals
- make the game playable without deployment
- support testing from multiple seat perspectives on one machine
- establish the server-authoritative platform core
- support the core Diplomacy game loop before adding structured diplomacy tracking

### Scope
- local-first runtime
- standard ruleset only
- standard map only
- game creation and initialization
- seat model
- admin/testing workflow for a single operator
- seat switching, impersonation, or equivalent multi-seat testing support
- order entry
- order revision before resolution
- adjudication integration
- phase advancement
- universal press
- direct two-seat communication

### Out of scope for this phase
- AI players
- deployment
- production auth
- public multiplayer operations
- structured diplomacy tracking beyond normal messaging

### Completion standard
MVP-A is complete when one person can run a full local game through multiple seat views using the admin/testing workflow.

---

## Phase 2 — Platform MVP-B: Structured Diplomacy Layer

### Purpose
Add a non-binding diplomacy-tracking layer that helps both humans and future AI players keep track of understandings, commitments, and later apparent follow-through.

### Goals
- preserve diplomatic history in a more useful form than raw chat alone
- support tracking of currently active understandings
- support tracking of prior understandings and later apparent honor/break behavior
- keep this system clearly outside adjudication

This means simple recorded apparent honor/break status is in scope. Derived trust scoring, reputation systems, or behavioral analytics are out of scope for the platform MVP.

### Scope
- structured diplomacy or commitment-tracking layer
- active vs historical diplomatic records
- commitments made and commitments received
- visibility rules for these records
- display/query surfaces in seat views and admin/debug views

### Important intentional non-decisions
The following are **not finalized at this stage**:
- exact terminology
- exact object model
- exact UX
- exact degree of structure vs free text
- exact treatment of honor/break status

### Hard rule
This layer is non-binding and must not be enforced by the adjudicator.

### Completion standard
MVP-B is complete when the playable local platform also supports a usable, persistent, non-binding diplomacy-tracking layer.

---

## Phase 3 — Platform MVP-C: AI-Pluggable Hardening

### Purpose
Harden the platform so that future AI players can plug into it without platform redesign.

### Goals
- make seat-facing platform behavior stable and machine-usable
- expose clean boundaries between platform and future AI runtime
- improve observability, serialization, and replayability

### Scope
- stable seat-facing interfaces
- machine-friendly state serialization
- event log and replay model
- clear and testable visibility rules
- durable access to diplomatic history
- explicit platform/AI boundary docs
- improved testability around state transitions

### Completion standard
MVP-C is complete when the platform is not only playable by humans locally, but is also structurally ready to host future AI-controlled seats.

---

## Phase 4 — AI Subsystem Design

### Purpose
Design the future AI player system against a real platform rather than a hypothetical one.

### Goals
- define the AI subsystem cleanly
- specify its boundaries with the platform
- define the contracts needed for an AI seat
- plan model/provider abstraction without committing prematurely to one backend

### Expected outputs
Documents under:
- `docs/ai/overview/`
- `docs/ai/future/`

Likely topics:
- AI system scope
- platform/AI boundary
- seat interface
- context model
- negotiation memory
- model/provider abstraction
- agent lifecycle

### Notes
This phase is design work, not yet broad AI implementation.

---

## Phase 5 — AI Prototype

### Purpose
Implement the first LLM-backed AI seat against the real platform.

### Likely scope
- model/provider abstraction
- per-seat context handling
- order generation loop
- press generation
- diplomatic memory updates
- initial evaluation harness

### Notes
This phase begins only after the platform is ready to host agent-controlled seats.

---

## Ruleset and map model

The platform should be designed around a **ruleset → map** hierarchy.

### Ruleset
A ruleset defines game semantics such as:
- phase model
- adjudication semantics
- order behavior
- other rules-level assumptions

### Map
A map defines game data such as:
- provinces
- coasts
- adjacencies
- powers
- supply centers
- starting units
- starting ownership
- rendering metadata

### Current support target
For now, only the following need to be supported:
- ruleset: `standard`
- map: standard classic map under that ruleset

### Architectural requirement
Even though only one ruleset and one map are initially provided, the system should be designed so that:
- adding another map under an existing ruleset is straightforward
- adding another ruleset later is possible without tearing apart the platform architecture

This is an architectural goal, not a near-term implementation goal.

---

## Deferred concerns

The following are intentionally deferred until later phases:

- production deployment
- production authentication
- matchmaking
- rankings and ladders
- tournament support
- spectator mode
- notifications
- operational scaling
- non-standard ruleset implementation
- broad variant support beyond the initial architecture

These should not distort MVP design.

---

## Near-term document sequence

The near-term documentation priority is:

1. `docs/overview/repository-roadmap.md`
2. `docs/overview/architecture-overview.md`
3. `docs/overview/terminology.md`
4. `docs/platform/mvp/01-product-scope.md`
5. `docs/platform/mvp/02-non-goals.md`
6. `docs/platform/mvp/03-domain-model.md`
7. `docs/platform/mvp/04-phase-state-machine.md`
8. `docs/platform/mvp/05-variant-schema.md`
9. `docs/platform/mvp/06-press-and-diplomacy-artifacts.md`
10. `docs/platform/mvp/07-local-dev-and-seat-testing.md`
11. `docs/platform/mvp/08-adjudication-boundary.md`
12. `docs/platform/mvp/09-api-surface.md`
13. `docs/platform/mvp/10-ui-surfaces.md`
14. `docs/platform/mvp/11-implementation-plan.md`

This order may be adjusted slightly as design dependencies become clearer, but it reflects the intended direction.

---

## Near-term implementation sequence

Implementation should begin only after the core MVP platform docs are in place.

The intended early implementation sequence is:

1. repository/app structure
2. local dev workflow
3. ruleset/map loading foundation
4. game/session/seat scaffolding
5. phase model and state progression
6. adjudication boundary integration
7. order entry and submission
8. admin/testing workflow for multi-seat local use
9. universal and direct press
10. later MVP structured diplomacy layer
11. AI-pluggable hardening work

---

## Guiding principles

All roadmap phases should preserve the following principles:

- platform and AI remain separate subsystems
- adjudication remains a clean boundary
- seat visibility rules are strict
- local-first usability comes before deployment concerns
- data-driven map support is built in early
- the platform must become AI-pluggable, but AI implementation begins later
- durable docs come before broad Codex-driven implementation

---

## Summary

This repository is being built in a deliberate sequence:

1. establish the repository foundation
2. build the platform MVP in three stages:
   - MVP-A playable local platform
   - MVP-B structured diplomacy layer
   - MVP-C AI-pluggable hardening
3. design the AI subsystem against the finished platform seams
4. implement the first AI player prototype

This sequencing is intentional and should guide both documentation and future implementation work.
