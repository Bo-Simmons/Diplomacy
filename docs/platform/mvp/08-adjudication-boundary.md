# Adjudication Boundary

## Purpose

This document defines the platform-level boundary between the Diplomacy platform and the adjudicator.

It exists to:
- keep rules resolution separate from messaging, UI, access, persistence, and future AI logic
- define what rules-relevant data enters adjudication
- define what normalized result data comes back to the platform
- constrain implementation without requiring a specific deployment style

This is an architecture/interface document. It is not an API transport spec, database schema, external engine choice, UI explanation design, or AI runtime design.

---

## Boundary Principles

The adjudicator resolves rules consequences only.

The platform orchestrates everything around adjudication:
- game creation
- phase lifecycle
- seat access
- order collection
- submission locking
- persistence
- reveal behavior
- snapshots
- event history
- read models
- next-phase creation

The adjudicator consumes rules state, not product state.

The adjudicator returns results to the platform. It must not directly mutate platform storage, advance phase lifecycle, create snapshots, record events, send messages, update read models, or create the next phase.

The platform is server-authoritative. Seat clients, admin/testing views, and future AI seats may request or submit actions, but the platform owns final accepted state and adjudication invocation.

---

## What The Adjudicator Receives

The adjudicator should receive only rules-relevant input.

Conceptual input set:
- ruleset reference
- map reference
- current resolved board state
- current phase kind
- adjudication context for that phase
- finalized effective submissions for that phase
- current obligations relevant to that phase
- retreat requirements, for retreat resolution
- adjustment requirements, for adjustment resolution

Ruleset and map input should include the rules-relevant authored data needed to resolve the phase:
- phase kind semantics
- powers
- playable provinces
- board locations
- land adjacency
- fleet adjacency
- supply-center data
- home-center data where needed for adjustment legality

Current resolved board state should use board locations for unit positions and provinces for province-level concepts such as supply-center control.

Finalized effective submissions are the platform-selected current submissions for the phase. Drafts and superseded submissions are not adjudicator input.

---

## What The Adjudicator Must Not Receive

The adjudicator must not receive product, UI, access, messaging, or AI state that is not needed for rules resolution.

Excluded input includes:
- messages
- commitment records
- diplomacy artifacts
- users
- sessions
- auth data
- admin/testing access data
- UI state
- read models
- non-final order drafts
- superseded submissions except where explicitly selected by the platform for rules-relevant audit input
- local-dev tokens
- model/provider configuration
- AI memory or planning state
- general product metadata not needed for rules resolution

Diplomacy artifacts may be socially meaningful, visible, storable, and queryable, but they are not mechanically binding and must not affect adjudication.

---

## What The Adjudicator Returns

The platform boundary expects a full normalized result object.

The adjudicator may compute internally using deltas, but the platform-facing result should be sufficient for the platform to persist the resolved state without reconstructing correctness from partial changes.

Conceptual output set:
- resulting resolved board state
- per-submission or per-order resolution details
- generated retreat requirements
- generated adjustment requirements
- supply-center control consequences where relevant
- unit creation, movement, dislodgement, removal, build, or disband outcomes
- invalid or failed order explanations where rules-relevant
- optional structured adjudication trace or explanation

The result may include both board-location-level and province-level data:
- unit positions are board-location based
- retreats and movement outcomes are board-location based
- supply-center control is province based
- home-center and build eligibility checks are province based

The platform owns how this result is persisted, revealed, projected into read models, and represented in history.

---

## Platform Ownership Around Adjudication

The platform owns all orchestration before and after the adjudicator is invoked.

Before adjudication, the platform owns:
- phase opening
- seat access checks
- input collection
- draft persistence
- submission persistence
- order replacement/revision rules before lock
- input locking
- selection of the effective submission per seat
- structural validation
- construction of the rules-relevant adjudication input

During adjudication, the platform owns:
- invoking the adjudicator
- handling adjudicator success or failure
- preserving server-authoritative control over accepted state

After adjudication, the platform owns:
- persistence of adjudication results
- updating current canonical state
- creating resolved phase snapshots at `resolved_revealed`
- event/history recording
- reveal and visibility behavior
- read-model updates
- creating retreat or adjustment requirements from the result
- opening the next phase when appropriate
- marking the previous phase `closed` when the next phase opens

The adjudicator does not decide visibility, send notifications, advance the phase lifecycle, or create the next phase.

---

## Validation Split

Validation should be split between platform structural validation and adjudicator rules validation.

### Platform Structural Validation

Platform-side structural validation may include:
- submission belongs to the correct game
- submission belongs to the correct phase
- submission belongs to the submitting seat
- submission is current or eligible to become effective before lock
- order shape is structurally valid
- referenced unit ids exist
- referenced board location ids exist
- referenced province ids exist where applicable
- referenced units belong to the submitting seat where applicable
- submitted action type is allowed for the phase kind at a structural level
- duplicate or missing required order lines are detected where this is structural rather than rules-semantic

Structural validation should reject malformed or mis-scoped input before adjudication.

### Adjudicator Rules Validation

The adjudicator owns rules legality and final resolution semantics.

Adjudicator-side validation and resolution includes:
- order legality under the ruleset
- movement legality
- support semantics
- convoy semantics
- bounce resolution
- cutting support
- dislodgement
- retreat consequences
- build and disband legality
- supply-center control consequences
- final success or failure of orders

The platform should not duplicate full rules legality in product code. It may provide helpful early structural feedback, but final legality and resolution belong to the adjudicator.

---

## Result Granularity

The adjudication boundary spans both board-location-level and province-level concepts.

Board-location-level data includes:
- unit positions
- movement sources
- movement targets
- retreat options
- dislodgement locations
- build locations where a coast or land location matters

Province-level data includes:
- supply-center status
- supply-center control
- home-center relationships
- victory threshold evaluation inputs

This mixed granularity is expected. The domain model treats units as occupying board locations while supply-center and home-center concepts remain province based.

---

## Trace And Explanation Expectations

A structured adjudication trace is useful for:
- admin/debug tooling
- implementation verification
- explaining outcomes
- regression testing
- future audit and replay work

Trace output is optional at the platform boundary.

Correctness must not depend on rich trace output. The platform should be able to persist and advance from the normalized adjudication result even when trace output is absent or minimal.

The trace should not become a second source of truth for game state.

---

## Deployment And Style Neutrality

The adjudicator may be implemented as:
- in-process platform code
- a wrapper around an existing adjudication engine
- an isolated service behind a boundary

The platform should depend on the adjudication interface, not on the deployment style.

The boundary should remain stable enough that the implementation style can change without mixing adjudication with messaging, UI, access, persistence, or future AI logic.

---

## Scope Boundary

This document does not define:
- final implementation stack
- final transport or API shape
- final database schema
- exact external adjudication engine choice
- detailed Diplomacy rules explanations
- future AI runtime behavior
- UI explanation system
- full event taxonomy

Detailed rule mechanics belong in the adjudicator implementation or adjudicator-specific tests. Platform docs should preserve the boundary and describe the data contract at an architecture level.
