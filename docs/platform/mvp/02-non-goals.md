# Platform MVP Non-Goals

## Purpose

This document defines what is intentionally out of scope for the platform MVP.

It exists to prevent scope creep, clarify deliberate deferrals, and distinguish **not now** from **never**. Some items listed here are expected later, but they should not be implemented as part of the current platform MVP unless explicitly re-scoped.

---

## Explicit Non-Goals

The platform MVP does not include:

- AI-player implementation
- model/provider abstraction implementation
- prompt systems, AI runtime loops, or AI memory systems
- deployment or cloud hosting
- production authentication
- public matchmaking
- rankings, ladder, or tournament systems
- spectator mode
- mobile-first UX
- polished moderation or operations tooling
- non-standard ruleset implementation
- shipping alternate map content beyond the standard map
- mechanically binding diplomacy agreements
- adjudicator enforcement of Agreement, Pact, Treaty, or other diplomacy artifacts
- advanced trust, betrayal, reputation, or behavioral analytics
- broad product polish beyond what local MVP usability requires

---

## Deferred But Architecturally Anticipated

Some out-of-scope items are known future directions. Current design should leave clean seams for them without implementing them now.

These include:

- alternate maps under rulesets
- future alternate rulesets
- AI-controlled seats
- production multiplayer concerns
- richer diplomacy-tracking semantics
- stronger operations, moderation, and deployment workflows

These are anticipated architecture pressures, not current implementation targets.

---

## Scope-Protection Guidance

Current implementation work should build the platform MVP directly.

Do not overbuild toward deferred features. Preserve boundaries where the long-term direction is already known, especially:

- platform vs future AI runtime
- adjudication vs messaging and diplomacy history
- gameplay rules vs admin/testing access
- standard rules logic vs data-driven map/setup definitions
- normalized domain entities vs seat/admin read models

The right MVP behavior is a clean local platform that can later be extended, not a partially implemented version of every future system.
