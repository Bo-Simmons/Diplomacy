# Platform MVP Product Scope

## Purpose

The platform MVP defines the complete platform milestone that must exist before AI-player implementation begins.

The goal is to build a local-first foundation for standard Diplomacy that can support human-operated seats now and future AI-controlled seats later. Current work should implement and harden the platform, not the AI-player subsystem.

This document is the authoritative scope anchor for the platform MVP.

---

## Product Definition

The platform MVP is a **local-first, server-authoritative Diplomacy web app**.

It supports:
- standard Diplomacy rules only
- the standard Diplomacy map only
- one-person local testing through admin/testing workflows
- multiple seat perspectives on one machine
- full core gameplay through standard phases
- universal press
- direct two-player press
- non-binding structured diplomacy history later in the MVP
- enough stable platform structure for future AI seats to plug in without platform redesign

The MVP is not merely the earliest playable slice. It is the full platform milestone made up of MVP-A, MVP-B, and MVP-C.

---

## Included Scope

The platform MVP includes:

- local runtime
- game creation and setup for the supported ruleset/map
- standard phase progression
- explicit seat model
- one-person admin/testing workflow
- multiple seat perspectives on one machine
- order drafting
- order revision before lock
- order submission
- adjudication integration behind a clean boundary
- phase lifecycle handling
- visibility-aware seat views
- admin/testing views separate from gameplay views
- universal press
- direct press between exactly two seats
- message history linked to phases
- game history sufficient for replay, debugging, and auditability
- current canonical state distinct from resolved snapshots and event history
- resolved phase snapshots at stable resolved/revealed boundaries
- later-in-MVP non-binding structured diplomacy or commitment tracking
- platform hardening sufficient for future AI seats to plug in without redesign

---

## Internal MVP Staging

The platform MVP is internally staged into three platform milestones.

### MVP-A: Playable Local Platform

MVP-A establishes the playable local foundation.

It includes local game creation, standard rules/map setup, seat modeling, order entry, order revision, order submission, adjudication integration, phase advancement, universal press, direct press, and a one-person admin/testing workflow for multiple seat perspectives.

MVP-A is complete when one person can run a full local standard game through multiple seat views using the admin/testing workflow.

### MVP-B: Structured Diplomacy Layer

MVP-B adds a persistent, non-binding diplomacy-tracking layer on top of platform communication.

This layer should help humans and future AI players track diplomatic understandings, commitments, and relevant history. It remains part of the communication layer and must not be enforced by the adjudicator.

The final terminology and detailed commitment model are intentionally deferred.

MVP-B is complete when the playable local platform also supports usable, persistent, visibility-aware structured diplomacy history.

### MVP-C: AI-Pluggable Hardening

MVP-C hardens the platform so future AI-controlled seats can be added without platform redesign.

It includes stable seat-facing behavior, clear visibility rules, durable access to game and diplomacy history, machine-usable read models, event/replay support, and explicit platform/AI boundaries.

MVP-C is complete when the platform remains fully playable by humans locally and is structurally ready to host future AI-controlled seats.

---

## Completion Standard

The platform MVP is complete when all of the following are true:

- one human operator can run a full local standard Diplomacy game across multiple seat perspectives
- the game loop works correctly through all standard phase types and lifecycle stages
- order drafting, revision, submission, adjudication, retreats, adjustments, and phase advancement are supported
- universal press and direct two-seat press work
- structured diplomacy history exists as a non-binding communication-layer feature
- seat-scoped visibility is preserved in normal seat views
- admin/testing access is explicit and separate from gameplay rules
- current canonical state, resolved phase snapshots, and event history are preserved as distinct concepts
- the adjudication boundary is clean and separate from messaging, UI, admin/testing access, and future AI runtime
- future AI-controlled seats can be added through platform-defined seat-facing interfaces without redesigning the platform core

---

## Scope Boundaries

The platform MVP does not include AI-player implementation.

It should preserve the platform/AI boundary by defining stable platform behavior, visibility rules, history models, and seat-facing contracts. It should not implement model/provider integrations, AI memory, negotiation engines, prompt systems, or AI decision loops.

Detailed implementation sequencing belongs in implementation-plan docs. Tooling, database design, API design, and final commitment-system terminology belong in later focused documents.
