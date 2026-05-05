# Tooling And Implementation Shape

## Purpose

This document records the chosen tooling and implementation shape for the platform MVP.

The product, architecture, domain model, phase model, variant schema, and adjudication boundary docs remain the source of truth for platform behavior. This document exists to keep scaffolding and implementation work from guessing the stack, repository shape, or broad application structure.

The current implementation target is the platform MVP. The future AI subsystem should influence clean boundaries, but it is not being implemented as part of this document.

## Chosen Stack

The platform MVP should use the following stack direction:

- Backend runtime: Go.
- Secondary language: Python where it is the practical fit.
- Frontend: TypeScript, React, and Vite.
- Database: PostgreSQL.
- Local orchestration: Docker Compose.
- Repository shape: monorepo.
- Application shape: modular monolith first.

These choices should guide initial scaffolding and implementation decisions. They do not define every framework, library, folder, migration tool, or internal package boundary.

## Repository And Application Shape

The repository should remain a monorepo containing the platform subsystem, future AI subsystem space, and shared documentation/contracts where useful.

For the platform MVP, the expected application shape is:

- one backend application for the platform runtime
- one frontend application for seat views and admin/testing views
- shared contracts or schemas where they reduce ambiguity between backend and frontend
- first-class ruleset and map data rather than standard-board hardcoding
- room for Python tools, adapters, or supporting services without forcing immediate runtime fragmentation

Do not lock exact folder names from this document alone. The first scaffolding pass should choose clear names that match the repository's established documentation structure and preserve platform/AI separation.

## Backend Guidance

Go is the primary platform runtime language.

The backend should be a modular monolith first. It should keep platform concerns separated internally while still shipping as a simple local application during the MVP.

Backend structure should be organized around platform responsibilities, such as:

- game/session orchestration
- phase lifecycle
- orders and submissions
- adjudication boundary
- ruleset and map references
- current canonical state
- phase snapshots and event history
- messages and diplomacy records
- seat-visible and admin/testing read models

Transport handlers should not dictate the core design. Request handling should call into explicit application/domain services rather than becoming the place where game behavior, persistence details, and visibility rules are mixed together.

The adjudicator must remain behind a formal boundary. Whether adjudication is implemented directly in Go, wrapped from another engine, or adapted later through another process is a boundary decision, not a reason to blend rules resolution into unrelated platform modules.

## Python Guidance

Python is an accepted secondary language and should not be artificially excluded.

Python is welcome where it is the most practical fit, including:

- ruleset/map validation tooling
- data import, normalization, or generation tools
- authored variant-data checks
- adjudication adapters where appropriate
- replay, debugging, or data transformation utilities
- future AI-adjacent utilities that are outside the primary platform runtime

The platform runtime should still remain coherent. Python should be used deliberately where it improves maintainability or practicality, not as accidental fragmentation of core platform behavior.

## Frontend Guidance

The frontend should be a TypeScript React application built with Vite.

The MVP frontend must support:

- seat-scoped gameplay views
- admin/testing views for one-person local operation
- multiple seat perspectives on one machine
- visibility-aware presentation of orders, messages, history, and pending actions

Frontend code should consume explicit backend contracts. It should not become the owner of domain rules, adjudication behavior, phase lifecycle semantics, or authoritative visibility decisions.

## Persistence Guidance

PostgreSQL is the primary persistence store for the platform MVP.

It is expected to persist platform concepts such as:

- current canonical state
- resolved phase snapshots
- event/history records
- messages
- diplomacy records or commitment-tracking data
- ruleset/map references
- seat, game, phase, order, and visibility-related records

This document does not define the final database schema. Persistence design should follow the domain model and architecture docs, especially the distinction between current canonical state, resolved snapshots, event history, core entities, and read models.

## Local Development

Docker Compose is the canonical local-stack path for the platform MVP.

The local stack should make it straightforward to run the backend, frontend, PostgreSQL, and any required supporting process for local development and one-person seat testing.

Direct local runs may still exist for convenience, especially during development, but Compose is the standard baseline that future setup instructions and agent tasks should be able to rely on.

## Realtime Guidance

The platform should be designed to become realtime-capable.

The MVP should not overbuild live collaboration infrastructure before the first playable local platform exists. Early implementation should preserve a clean path for updating seat and admin/testing views as phase, order, message, and history state changes, without forcing a premature realtime architecture.

## Coding Style Direction

Implementation work should follow `.local/style-guide.md` when that file exists in the repository root. That file is intentionally local-only and should not be copied into committed documentation.

At a high level, platform code should favor:

- readability
- explicit naming
- clear module boundaries
- maintainable code over cleverness
- comments used deliberately to clarify intent, flow, and non-obvious decisions
- a traditional backend-engineering style rather than terse or overly smart code

These preferences should guide implementation style without exposing or inventing the detailed contents of the local guide.

## Boundaries To Preserve

Implementation work must preserve the already-locked platform boundaries:

- platform and future AI runtime are separate subsystems
- adjudication is separate from messaging, UI, persistence orchestration, and future AI logic
- domain logic is separate from transport and persistence details
- read models are separate from normalized core entities where appropriate
- seat visibility is enforced by platform-side rules and contracts
- admin/testing access does not change gameplay rules
- ruleset/map data remains first-class and variant-ready

These boundaries are more important than short-term convenience in scaffolding.

## Scope Boundary

This document does not define:

- exact folder-by-folder repository layout
- final API surface
- final database schema
- exact Go router or framework choice
- exact ORM, query, or migration library
- exact frontend state-management library
- final CI/CD setup
- production deployment architecture
- future AI runtime tooling
- AI subsystem implementation

The intent is to constrain the platform MVP's implementation shape without turning tooling choices into a detailed implementation plan.
