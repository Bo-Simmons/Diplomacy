# Implementation Plan

## Purpose

This document defines the recommended build sequence for the platform MVP.

The architecture docs define what the platform is. This document translates those decisions into a practical implementation path for Codex tasks and human review.

The current build target is the platform MVP: a local-first, server-authoritative Diplomacy platform with standard rules/map support, strict seat-scoped visibility, clean adjudication boundaries, admin/testing workflows, press, structured diplomacy history, and enough platform-side contract stability for future AI seats.

This is an implementation sequencing document. It is not a ticket backlog, final API spec, final database schema, or final repository layout.

## Implementation-Plan Principles

- Top-level implementation phases are capability-driven.
- Each top-level phase should be small enough to plausibly map to a PR-sized milestone.
- A phase should deliver one meaningful vertical capability, not only a layer of infrastructure.
- Commits inside a phase may be dependency-ordered or grouped by concern.
- A commit does not need to be independently product-complete.
- Commits should still be coherent, reviewable, and easy to reason about.
- Implementation should move quickly from scaffold to integrated, testable slices.
- Avoid prolonged infrastructure-only work after the foundation phase.
- Do not let transport handlers, framework conventions, or persistence details become the architecture.
- Keep domain logic separate from transport, persistence, UI, and read-model shaping.
- Preserve the adjudication boundary, seat visibility boundary, and platform/future-AI boundary from the start.

## Phase/PR Versus Commit Guidance

Treat an implementation phase as a PR-sized capability milestone.

The phase should be understandable as a user-facing or operator-facing capability, such as "create a local game and view seats" or "submit orders and resolve through the adjudication boundary."

Commits within that phase may be organized by implementation dependency order. For example, a capability PR might contain commits shaped like:

- docs/contracts
- schema/data
- backend/domain
- transport/handlers
- frontend
- tests/validation
- cleanup

That is acceptable. The important rule is that the top-level phase delivers an integrated capability by the end.

## Recommended Phase Sequence

The recommended platform MVP build sequence is:

1. Phase 0: foundation and scaffold
2. Phase 1: ruleset/map loading and reference data
3. Phase 2: game creation, seats, and current-state shell
4. Phase 3: phase lifecycle, snapshots, and event history
5. Phase 4: orders and adjudication boundary integration
6. Phase 5: local admin/testing workflow and seat-scoped views
7. Phase 6: press
8. Phase 7: structured diplomacy layer
9. Phase 8: MVP-C hardening / AI-pluggable platform surface

Phases 0 through 6 build the playable local platform foundation. Phase 7 adds the structured diplomacy layer. Phase 8 hardens the platform-facing contracts and operational shape for future AI-seat integration without implementing the AI subsystem.

## Phase 0: Foundation And Scaffold

### Goal

Create the basic monorepo application scaffold so the platform can be built, run, and tested locally without guessing the stack.

Phase 0 may be more infrastructure-oriented than later phases, but it should still end with a runnable local skeleton.

### Done At End

- Go backend application skeleton exists.
- TypeScript React Vite frontend skeleton exists.
- PostgreSQL is available through Docker Compose.
- Local stack can boot through the standard local path.
- Backend exposes a minimal health/status path or equivalent smoke-test surface.
- Frontend can reach the backend in local development.
- Basic test/validation commands exist for backend and frontend.
- The scaffold leaves clear room for platform modules, ruleset/map data, and future Python tooling.

### Suggested Workstreams

- Backend: create the Go application entrypoint, configuration loading, logging shape, and minimal app wiring.
- Frontend: create the React/Vite application shell and local backend connectivity.
- Data: add local database service configuration without defining the final schema.
- Python/tooling: reserve a practical place for validation or data tooling if needed, but do not invent tools yet.
- Tests/validation: add smoke checks for local boot, backend test command, and frontend validation command.

### Out Of Scope

- Full game creation.
- Final API design.
- Final database schema.
- Production deployment.
- AI subsystem implementation.

### Dependencies/Notes

Follow the tooling and implementation-shape doc. Keep the scaffold modular-monolith friendly and avoid framework-first domain design.

## Phase 1: Ruleset/Map Loading And Reference Data

### Goal

Load and validate the standard ruleset and standard map as first-class reference data, without hardcoding the board into game logic.

### Done At End

- Standard ruleset and standard map reference data can be loaded locally.
- Ruleset owns map definitions conceptually.
- Province ids, board-location ids, powers, supply centers, home centers, starting units, starting supply-center control, adjacencies, and map features are represented according to the variant schema.
- Land and fleet adjacency data are separate.
- Convoy reachability is not stored as a third graph.
- Validation catches major authored-data invariants.
- A simple admin/debug endpoint or local tool can confirm loaded reference data.

### Suggested Workstreams

- Data: author the initial standard ruleset/map reference data.
- Backend: add loaders and typed reference-data structures.
- Python/tooling: add practical validation tooling if Python is the better fit.
- Frontend: optionally expose a minimal admin/reference-data inspection view.
- Tests/validation: cover map-data invariants, adjacency references, and starting-state references.

### Out Of Scope

- Game runtime state.
- Order submission.
- Adjudication.
- Alternate maps or alternate rulesets beyond preserving the data shape.

### Dependencies/Notes

Use the variant-schema doc as the source of truth. This phase should make future map additions plausible without designing a universal game-schema system.

## Phase 2: Game Creation, Seats, And Current-State Shell

### Goal

Create a local standard game with seats and an initial current canonical state shell.

### Done At End

- A local operator can create a supported standard game.
- Seats are created as the primary gameplay actor boundary.
- Initial units, supply-center control, powers, and current phase reference are initialized from standard map data.
- Current canonical state exists as the operational truth for the live game.
- Admin/testing view or backend surface can list games, seats, and initial state.
- Seat-scoped read boundaries are present even if the participant UI is still basic.

### Suggested Workstreams

- Backend: add game creation, seat creation, initial state creation, and current-state storage.
- Data: connect starting-state definitions to game initialization.
- Frontend: add minimal local game creation and game/seat list surfaces.
- Tests/validation: cover game initialization from reference data and seat ownership boundaries.

### Out Of Scope

- Full phase lifecycle.
- Order drafting/submission.
- Adjudication.
- Press.
- Structured diplomacy.

### Dependencies/Notes

Do not require replay of event history to know current state. Event history will arrive in a later phase, but current canonical state should already be the live operational model.

## Phase 3: Phase Lifecycle, Snapshots, And Event History

### Goal

Implement the platform-owned phase lifecycle and the three distinct state/history concepts: current canonical state, resolved phase snapshots, and event history.

### Done At End

- Phase kind and phase stage are represented separately.
- Standard phase kinds and transitions are supported.
- Platform-standard stages exist: `open_for_input`, `input_locked`, `resolving`, `resolved_revealed`, and `closed`.
- A phase can become `resolved_revealed` and remain current until the next phase opens.
- A phase becomes `closed` when the next phase opens.
- Resolved phase snapshots are taken at stable resolved/revealed boundaries.
- Event history records major lifecycle events without replacing first-class domain entities.
- Admin/testing surfaces can inspect current phase, phase stage, snapshots, and event history.

### Suggested Workstreams

- Backend: add phase lifecycle services, transition rules, snapshot creation, and event recording.
- Data: represent standard-ruleset phase cycle metadata.
- Frontend: expose current phase/stage and history inspection in admin/testing views.
- Tests/validation: cover transition rules, conditional phase placeholders, snapshot boundaries, and closed-phase behavior.

### Out Of Scope

- Complete order resolution.
- Press policy details beyond phase/stage placeholders.
- Full replay system.
- AI-facing history contracts.

### Dependencies/Notes

This phase should implement platform phase orchestration without building a generic workflow engine or scripting system.

## Phase 4: Orders And Adjudication Boundary Integration

### Goal

Allow seats to draft, revise, submit, lock, and resolve orders through the explicit adjudication boundary.

### Done At End

- Seat-scoped order drafting exists for phase-appropriate actions.
- Seats can revise orders before lock/submission cutoff.
- The platform selects effective finalized submissions for the phase.
- Platform-side validation covers structural validation only.
- Rules legality and resolution semantics are delegated to the adjudicator boundary.
- The adjudicator receives rules-relevant input and returns a normalized result object.
- Current canonical state, snapshots, events, retreat/adjustment requirements, and reveal behavior are updated by the platform around adjudication.
- Full finalized submissions become visible at `resolved_revealed`.

### Suggested Workstreams

- Backend: implement order submission model, effective submission selection, structural validation, adjudicator interface, and result application.
- Data: connect orders to board-location ids and phase kinds.
- Frontend: add basic seat order drafting/submission UI and admin submission status.
- Python/tooling: consider an adjudication adapter or validation helper only if it is the practical fit.
- Tests/validation: cover structural validation, adjudicator input/output contract, result application, reveal timing, and phase consequences.

### Out Of Scope

- Messaging.
- Structured diplomacy.
- Rich adjudication explanation UI.
- Choosing a final external judge deployment style.
- Future AI order generation.

### Dependencies/Notes

The adjudicator must not receive messages, users, sessions, admin/testing access data, UI state, read models, non-final drafts, or product metadata that is not rules-relevant.

## Phase 5: Local Admin/Testing Workflow And Seat-Scoped Views

### Goal

Make the platform practically testable by one operator across multiple real seat-scoped views.

### Done At End

- Admin/testing mode can create, start, inspect, and reset local games.
- Admin/testing mode can inspect seats, phase/stage, submission status, adjudication results, current state, snapshots, and event history.
- Admin/testing mode can open or generate separate seat-scoped views.
- Seat views are backed by real seat-scoped backend reads.
- Seat views show only visible state, messages/channels when available, pending actions, and allowed actions for that seat.
- Simplified local access exists without collapsing seat visibility.
- One operator can run a local game across multiple seat perspectives.

### Suggested Workstreams

- Backend: add admin/testing read models, seat-visible read models, and simplified local access mechanics.
- Frontend: build admin/testing surface and participant seat view shell.
- Tests/validation: cover visibility boundaries and local multi-seat workflow.
- Data: support reset/fresh-game flows against the standard start state.

### Out Of Scope

- Production authentication.
- Public multiplayer.
- Spectator product design.
- Polished final UI.
- Inline admin impersonation as a required feature.

### Dependencies/Notes

Separate seat-scoped views are the canonical testing path. Admin/testing access is explicit and separate from gameplay rules.

## Phase 6: Press

### Goal

Add first-class platform messaging for universal press and direct two-seat press.

### Done At End

- Messages are first-class communication objects.
- Messages are phase-linked.
- Universal press is visible to all seats.
- Direct press is visible only to the two participating seats.
- Seat-scoped views show only seat-visible messages and channels.
- Admin/testing views can inspect press for local debugging through explicit admin/testing surfaces.
- Message sending is reflected in event history without reducing messages to generic event rows.

### Suggested Workstreams

- Backend: add message/channel model, visibility rules, phase linkage, and message events.
- Frontend: add universal and direct two-seat press UI in seat views and inspection support in admin/testing views.
- Tests/validation: cover visibility for universal and direct press, phase linkage, and event-history recording.

### Out Of Scope

- Structured diplomacy records.
- Production moderation tooling.
- Notifications.
- Public spectator chat.
- AI negotiation behavior.

### Dependencies/Notes

Keep adjudication separate from messaging. Press visibility should exercise the same seat boundary future AI seats will use.

## Phase 7: Structured Diplomacy Layer

### Goal

Add the MVP-B structured diplomacy / commitment-tracking layer on top of first-class messages.

### Done At End

- The platform can record non-binding structured diplomacy records or commitments.
- Tracked commitments link back to specific messages.
- Structured diplomacy visibility follows communication visibility rules.
- Apparent honor/break status can be recorded or structurally tracked at a simple level.
- Advanced trust, betrayal, reputation, scoring, or behavioral analytics remain out of scope.
- The adjudicator does not enforce commitments.
- Exact terminology and taxonomy remain intentionally provisional unless later docs lock them.

### Suggested Workstreams

- Backend: add commitment/diplomacy record model, message linkage, visibility rules, and simple status handling.
- Frontend: add minimal creation/inspection UI tied to messages.
- Tests/validation: cover message linkage, visibility, non-binding behavior, and simple status changes.

### Out Of Scope

- Mechanically binding agreements.
- Final commitment-system taxonomy.
- Advanced trust or reputation analytics.
- AI memory or negotiation systems.
- Judge enforcement of diplomacy records.

### Dependencies/Notes

This phase builds on press. It should remain a communication-layer capability, not a rules-engine feature.

## Phase 8: MVP-C Hardening / AI-Pluggable Platform Surface

### Goal

Harden the completed local platform so future AI-controlled seats can plug into platform-defined contracts without platform redesign.

This phase does not implement AI players.

### Done At End

- Seat-visible state/read models are stable enough to be consumed consistently.
- Legal action surfaces are explicit for orders and phase-appropriate seat actions.
- Message and history visibility obey seat boundaries through defined read contracts.
- Phase, stage, action, reveal, and submission lifecycle expectations are documented and exercised.
- Event/history and snapshots are usable for debugging, replay, and later AI-facing history use.
- Admin/testing workflows remain explicit and local-first.
- Platform/future-AI boundaries are preserved.

### Suggested Workstreams

- Backend: refine seat-facing contracts, legal action descriptions, lifecycle surfaces, and visibility enforcement.
- Frontend: smooth seat/admin workflows where needed for local usability.
- Tests/validation: add contract-style tests for seat-visible state, legal actions, message/history visibility, and lifecycle transitions.
- Python/tooling: add replay or validation utilities if practical and clearly useful.
- Docs/contracts: update focused docs if implementation reveals ambiguity.

### Out Of Scope

- AI player implementation.
- Model/provider abstraction.
- Prompt systems.
- AI memory or negotiation runtime.
- Production deployment.
- Public multiplayer operations.

### Dependencies/Notes

This phase proves the platform side of AI readiness. It should make future AI integration possible through stable seat-facing behavior, not by adding AI runtime code.

## Cross-Phase Review Checks

Every implementation phase should preserve these checks:

- Platform remains local-first.
- Docker Compose remains the canonical local-stack path.
- Go remains the primary platform runtime.
- Python is allowed where it is the practical fit, but core runtime behavior should not fragment unnecessarily.
- Frontend remains TypeScript React with Vite.
- PostgreSQL remains the primary persistence store.
- Domain logic stays separate from transport, persistence, and UI.
- Adjudication stays behind a formal boundary.
- Seat-scoped visibility is enforced by backend reads and contracts.
- Admin/testing access remains explicit and separate from gameplay rules.
- Current canonical state, snapshots, and event history remain distinct.
- Read models remain separate from core domain entities where appropriate.
- Deferred AI concerns do not become platform implementation work.

## Scope Boundary

This document does not define:

- a complete task backlog
- a project-management template
- exact PR boundaries for every repository change
- final API endpoints
- final database schema
- final folder-by-folder repository layout
- detailed UI mockups
- exact framework/library choices inside the chosen stack
- AI subsystem implementation

Use this plan as the implementation sequence for future Codex tasks, then keep each task narrowly scoped to the capability phase it is advancing.
