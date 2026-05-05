# Local Development And Seat Testing

## Purpose

This document defines the local development and testing model for the platform MVP.

The MVP must be fully testable by one human operator on one machine. Local testing must remain faithful to real gameplay behavior, especially seat-scoped visibility, even when local access and admin controls are simplified for development.

This document is not a production operations guide. It describes how the local platform should be shaped so implementation work can support practical MVP development without weakening core platform boundaries.

## Core Decisions

- The platform MVP is local-first.
- One-operator multi-seat testing is a required workflow.
- Admin/testing functionality is first-class MVP functionality, not a temporary hack.
- The primary testing model is an admin/testing view that opens separate seat-scoped views.
- Separate seat-scoped views are the canonical participant testing path.
- Inline admin impersonation is not required for the MVP.
- Seat views must be backed by real seat-scoped backend reads.
- Local access may be simplified, but it must not collapse gameplay visibility boundaries.
- The local stack should support both fast fresh-game setup/reset and persistent inspection/debugging.

## Supported Local Workflows

The local MVP should support these workflows:

- boot the full local stack
- create a fresh supported standard game
- open multiple seat perspectives on one machine
- run a full game solo across multiple seats
- draft, revise, and submit orders from seat-scoped views
- send and inspect seat-visible press from seat-scoped views
- advance through standard phase kinds and lifecycle stages
- inspect current game state, phase state, submissions, adjudication results, snapshots, messages, and event history
- reset local game data when a fresh scenario is needed

The canonical local stack path is defined by the tooling document. This document defines the required behavior and workflow shape, not exact commands.

## Local Operating Modes

### Normal Local App Mode

Normal local app mode is the ordinary locally running platform application.

It should exercise the same core backend behavior as other environments: game state is server-authoritative, phase lifecycle is platform-owned, adjudication stays behind its boundary, and seat visibility is enforced by backend read models and contracts.

Normal local app mode may use simplified local access while production authentication remains out of scope.

### Admin/Testing Mode

Admin/testing mode is an explicit local development surface with broader inspection and control capabilities.

It exists to let one operator create games, inspect progress, open seat views, debug state, and run full solo test games. It is part of the MVP product scope for local development and testing.

Admin/testing mode is not a gameplay rule and should not become a hidden path for gameplay actions that bypass seat constraints.

### Seat-Scoped View Mode

Seat-scoped view mode is the participant-facing view for a single seat.

Each seat view should behave as if the seat occupant is using the platform normally. It should show only that seat's visible state, messages, channels, pending actions, history, and phase-relevant controls.

Seat-scoped views are the primary way to test player-perspective behavior.

## Admin/Testing Mode Capabilities

Admin/testing mode should provide practical local controls for MVP development:

- create a new supported standard game
- start or reset local games
- inspect the seat list and seat status
- inspect the current phase, phase kind, and phase stage
- inspect order submission status by seat
- inspect adjudication results and generated phase consequences
- inspect current canonical state, resolved snapshots, messages, and event history
- open or generate separate seat-scoped views for each seat
- use explicit dev/debug controls where appropriate

Admin/testing read models may expose broader information than a seat can see. Those broader views must remain explicit admin/testing surfaces.

## Seat-Scoped View Model

Seat views are the canonical participant views.

A seat-scoped view should:

- show only seat-visible board state and pending actions
- show only seat-visible messages and channels
- show only diplomacy records visible to that seat
- allow only actions available to that seat in the current phase kind and stage
- reflect phase lifecycle rules faithfully
- reveal finalized phase submissions only when reveal policy allows them
- use real seat-scoped backend reads rather than filtering a full admin payload in the browser

This model is important for future human and AI seats. Future AI-controlled seats should be able to rely on the same seat-facing boundary that local human seat views use.

## Access Simplification Policy

Local MVP development may use simplified access models, such as:

- a local admin mode
- generated dev seat links
- generated dev seat tokens
- simplified local access flows

These mechanisms are developer convenience concerns. They must not redefine gameplay visibility.

Even in local-only mode:

- direct press remains visible only to its participating seats
- universal press remains visible to all seats
- seat-specific pending actions remain seat-specific
- private diplomacy records remain visibility-scoped
- admin/testing visibility remains separate from participant visibility
- seat-scoped backend reads remain real

If a simplified local access path makes it possible to open multiple seats quickly, each opened seat view must still receive data through the correct seat boundary.

## Preferred Seat Testing Model

The preferred MVP testing model is:

1. Open the admin/testing surface.
2. Create or select a local game.
3. Use admin/testing controls to open separate seat-specific views.
4. Test each seat through its own participant view.

Inline admin impersonation is not required. Separate seat views are preferred because they make visibility errors easier to catch and more closely simulate future human or AI seat behavior.

If inline impersonation is ever added later, it should remain an explicit admin/testing convenience and should still exercise the same seat-scoped backend read model.

## Practical Local Testing Patterns

The local workflow should be comfortable with ordinary browser usage patterns:

- one admin window plus several seat windows
- multiple tabs in one browser
- multiple browser profiles
- normal and private/incognito windows
- one seat per window when testing visibility-sensitive workflows

The platform should not require production-style account management for these local workflows.

## Reset And Debug Expectations

Local development should make it easy to move between clean starts and persistent debugging.

The local MVP should support:

- creating a fresh game from the supported standard start state
- resetting a local game or local data set when needed
- inspecting an existing game without destroying its history
- preserving event history for debugging and replay
- inspecting resolved phase snapshots at stable reveal boundaries
- comparing current canonical state against historical snapshots and events

Reset controls should be explicit local/admin actions. They should help development move quickly without obscuring the distinction between current canonical state, resolved snapshots, and event history.

## Scope Boundary

This document does not define:

- production authentication
- cloud deployment
- public multiplayer workflows
- public account management
- spectator product design
- full API surface
- final UI polish
- detailed UI mockups
- exact local command names

The goal is to constrain local MVP implementation enough that one operator can reliably test real seat-scoped gameplay on one machine.
