# Domain Model

## Purpose

This document defines the core domain model for the platform MVP.

It covers:
- reusable reference definitions
- runtime game entities
- orders and adjudication artifacts
- press and commitment history
- identity and access concepts
- read-model boundaries

It does not yet define:
- tooling choices
- final database schema
- final API payloads
- detailed AI player architecture

---

## Modeling principles

### 1. Seat is the core gameplay actor boundary
The platform should model game participation primarily through **seats**, not through users or players.

A seat corresponds to one power in one game.

### 2. Ruleset and map are separate concepts
A **ruleset** defines gameplay semantics.
A **map** defines board topology, powers, starting state, and rendering metadata.

Maps exist under rulesets.

### 3. Phase is explicit
The platform must model **Phase** as a first-class concept rather than relying on loose year/season strings.

The canonical phase type should distinguish at least:
- Spring Orders
- Spring Retreats
- Fall Orders
- Fall Retreats
- Winter Adjustments

### 4. Current state and history both matter
The platform should preserve:
- current canonical game state
- resolved phase snapshots
- event history

### 5. Orders are structured objects
Orders should be represented internally as structured data, not plain text blobs.

### 6. Communication is phase-linked
Messages and commitment-tracking artifacts should be associated with the phase in which they occur.

### 7. Commitments are made through messages
Tracked commitments should be linked to the specific message or messages through which they were created or acknowledged.

### 8. Read models are separate from core domain entities
Frontend/API views should not directly expose the normalized domain model.

---

## Domain model overview

The model is divided into six areas:

1. Reference definitions
2. Runtime game model
3. Orders and adjudication
4. Communication and commitment tracking
5. Identity and access
6. Read models and history

---

## 1. Reference definitions

These entities define reusable game data. They are not tied to one running game.

### RulesetDefinition
Defines gameplay semantics.

Candidate fields:
- ruleset_id
- name
- description
- phase_model
- supported_unit_types
- supported_order_types
- adjudication_semantics_version
- is_active

Notes:
- For now, only the standard ruleset is required.
- This should remain first-class even though only one ruleset exists initially.

---

### MapDefinition
Defines a map under a ruleset.

Candidate fields:
- map_id
- ruleset_id
- name
- description
- power_count
- victory_threshold
- render_metadata
- is_active

Notes:
- For now, only the standard classic map is required.

---

### PowerDefinition
Defines a power that exists on a map.

Candidate fields:
- power_id
- map_id
- name
- abbreviation
- color
- display_order

Notes:
- Powers are map-defined, not globally hardcoded.

---

### ProvinceDefinition
Defines a province on a map.

Candidate fields:
- province_id
- map_id
- name
- abbreviation
- province_type
- is_supply_center
- render_metadata

Where `province_type` is one of:
- inland
- coastal
- sea

---

### BoardLocationDefinition
Defines an occupiable/targetable board location.

This exists to model coasts cleanly.

Candidate fields:
- location_id
- map_id
- province_id
- location_type
- coast_name_optional
- unit_type_restrictions
- render_metadata

Where `location_type` may include:
- inland_location
- sea_location
- coastal_location
- named_coast_location

Notes:
- A province may have one or multiple board locations.
- Fleets interact with coast-aware locations.
- Armies may often interact at the province level, depending on the province.

---

### AdjacencyDefinition
Defines movement adjacency between board locations.

Candidate fields:
- adjacency_id
- map_id
- from_location_id
- to_location_id
- allowed_unit_types
- movement_kind

Notes:
- Adjacency should be modeled as a graph, not buried informally in province metadata.
- `movement_kind` may be useful later if the model distinguishes normal movement from special movement handling.

---

### StartingUnitDefinition
Defines a starting unit on a map.

Candidate fields:
- starting_unit_id
- map_id
- power_id
- unit_type
- location_id

---

### StartingSupplyCenterControlDefinition
Defines starting ownership/control of supply centers.

Candidate fields:
- starting_control_id
- map_id
- province_id
- power_id

---

## 2. Runtime game model

These entities represent a specific game in progress.

### Game
The root runtime entity for a match.

Candidate fields:
- game_id
- ruleset_id
- map_id
- status
- created_at
- started_at
- ended_at
- current_phase_id
- current_snapshot_id

Notes:
- A game references one ruleset and one map.
- Status may include values such as setup, active, completed, abandoned.

---

### GameConfig
Stores per-game operational configuration.

Candidate fields:
- game_config_id
- game_id
- advancement_mode
- phase_deadline_settings
- local_dev_mode_enabled
- admin_tools_enabled
- notes_optional

Notes:
- This keeps configuration separate from ruleset/map definitions.

---

### Seat
Represents one occupiable role in one game.

Candidate fields:
- seat_id
- game_id
- power_id
- status
- current_occupant_type_optional
- current_occupant_ref_optional

Notes:
- A seat corresponds to one power within one game.
- Seats are the core gameplay boundary.
- A seat may eventually be occupied by a human, AI, or nobody.

---

### Phase
Represents one explicit game phase.

Candidate fields:
- phase_id
- game_id
- year
- phase_type
- status
- opened_at
- locked_at
- resolved_at
- previous_phase_id_optional
- next_phase_id_optional

Where `phase_type` should be one of:
- spring_orders
- spring_retreats
- fall_orders
- fall_retreats
- winter_adjustments

Notes:
- This is the canonical time/state progression concept.
- It is more precise than season/year alone.

---

### PositionSnapshot
Represents a resolved board state at a specific phase boundary.

Candidate fields:
- snapshot_id
- game_id
- phase_id
- snapshot_kind
- created_at

Notes:
- The platform should preserve explicit resolved snapshots per phase.
- This should coexist with event history, not replace it.

---

### Unit
Represents a runtime unit in a game.

Candidate fields:
- unit_id
- game_id
- seat_id
- unit_type
- current_location_id
- status
- created_in_phase_id
- removed_in_phase_id_optional

Notes:
- Units should have stable identities across the game.
- Stable unit ids help with debugging, history, and later AI/state reasoning.

---

### SupplyCenterControl
Represents current control of a supply center in a game.

Candidate fields:
- supply_center_control_id
- game_id
- province_id
- controlling_seat_id
- effective_phase_id

Notes:
- This should be explicit, not inferred ad hoc from unrelated state.

---

### RetreatRequirement
Represents a retreat obligation created by adjudication.

Candidate fields:
- retreat_requirement_id
- game_id
- phase_id
- unit_id
- dislodged_from_location_id
- legal_retreat_location_ids
- status

---

### AdjustmentRequirement
Represents required builds or disbands for a seat.

Candidate fields:
- adjustment_requirement_id
- game_id
- phase_id
- seat_id
- net_adjustment
- legal_build_location_ids_optional
- status

Notes:
- Positive net adjustment means builds.
- Negative net adjustment means disbands.

---

## 3. Orders and adjudication

These entities cover order entry, submission, and resolution.

### OrderDraft
Represents a mutable saved draft of a seat’s orders for a phase.

Candidate fields:
- order_draft_id
- game_id
- phase_id
- seat_id
- status
- saved_at

Notes:
- Drafts allow revision before lock.

---

### OrderSubmission
Represents a submitted order set for a seat in a phase.

Candidate fields:
- order_submission_id
- game_id
- phase_id
- seat_id
- submitted_at
- replaced_submission_id_optional
- status

Notes:
- A phase may contain multiple submissions over time, but only one current effective submission per seat.

---

### OrderEntry
Represents one structured order line.

Candidate fields:
- order_entry_id
- order_submission_id
- unit_id
- order_type
- source_location_id
- target_location_id_optional
- auxiliary_unit_id_optional
- auxiliary_location_id_optional
- raw_notation_optional

Notes:
- The exact field set may evolve as order types are nailed down.
- The important point is that orders are structured internally.

---

### AdjudicationRun
Represents one adjudication execution for a phase.

Candidate fields:
- adjudication_run_id
- game_id
- phase_id
- input_snapshot_id
- started_at
- completed_at
- judge_version
- trace_optional
- status

Notes:
- This is the platform-facing boundary artifact for judge execution.

---

### ResolutionEntry
Represents an outcome associated with a specific order, unit, or resolution event.

Candidate fields:
- resolution_entry_id
- adjudication_run_id
- related_order_entry_id_optional
- related_unit_id_optional
- resolution_type
- summary
- detail_optional

Notes:
- Useful for UI explanation, debugging, and auditability.

---

## 4. Communication and commitment tracking

### Channel
Represents a press channel within a game.

Candidate fields:
- channel_id
- game_id
- channel_type
- status

Where `channel_type` is initially:
- universal
- direct

Notes:
- Direct channels will also need participant seat references.

---

### ChannelParticipant
Represents seat membership or participation in a channel.

Candidate fields:
- channel_participant_id
- channel_id
- seat_id

Notes:
- Universal channels may be treated specially, but the model should still be able to express visibility clearly.

---

### Message
Represents one message sent through a channel.

Candidate fields:
- message_id
- game_id
- phase_id
- channel_id
- sender_seat_id
- content
- created_at
- edited_at_optional
- superseded_message_id_optional

Notes:
- Messages are phase-linked.
- Messages are the origin point for tracked commitments.

---

### CommitmentRecord
Placeholder name for the later structured diplomacy-tracking layer.

This name is provisional.

Candidate fields:
- commitment_record_id
- game_id
- created_in_phase_id
- originating_message_id
- status
- title_optional
- body_summary_optional
- created_at
- closed_at_optional

Notes:
- Exact terminology and final model are intentionally still open.
- The important architectural decision is that commitments are created through messages and linked back to those messages.

---

### CommitmentParticipant
Represents a seat’s relationship to a commitment record.

Candidate fields:
- commitment_participant_id
- commitment_record_id
- seat_id
- role

Possible roles may later include:
- proposer
- recipient
- acknowledger
- participant

Notes:
- The final role set is still open.

---

### CommitmentLink
Represents secondary message links relevant to the same commitment.

Candidate fields:
- commitment_link_id
- commitment_record_id
- message_id
- link_type

Possible `link_type` values may later include:
- acknowledgment
- counter
- clarification
- cancellation
- follow_up

Notes:
- This preserves the fact that commitments live through message history rather than existing as detached abstractions.

---

## 5. Identity and access

These concepts are important, but they should remain outside the core game model where possible.

### User
Represents a human account identity.

Candidate fields:
- user_id
- display_name
- auth_provider_optional
- created_at
- status

Notes:
- Local MVP may use a simplified model here.

---

### UserSeatAccess
Represents a user’s access to a seat in a game.

Candidate fields:
- user_seat_access_id
- user_id
- game_id
- seat_id
- access_role
- granted_at

Possible access roles may later include:
- seat_operator
- admin
- observer

Notes:
- This keeps user identity separate from gameplay entities.

---

### DevSeatAccessToken
Optional local-dev concept for easy one-person multi-seat testing.

Candidate fields:
- dev_seat_access_token_id
- game_id
- seat_id
- token
- created_at
- expires_at_optional
- status

Notes:
- This is an application/access concern, not a game rule concept.

---

## 6. History, audit, and projections

### GameEvent
Represents an append-only event in game history.

Candidate fields:
- game_event_id
- game_id
- phase_id_optional
- event_type
- actor_type_optional
- actor_ref_optional
- occurred_at
- payload

Notes:
- This is for auditability and replay support.
- Not every projection should be reconstructed from events, but events should still be preserved.

---

### PhaseHistoryEntry
Represents a summarized historical record for a completed phase.

Candidate fields:
- phase_history_entry_id
- game_id
- phase_id
- snapshot_id
- adjudication_run_id_optional
- summary
- created_at

---

### SeatViewModel
A read projection for one seat.

This is not a core domain entity.
It is a backend-assembled view.

It should include:
- seat-visible board state
- current phase
- visible channels/messages
- seat orders/drafts/submissions
- visible commitment records
- local actionable items

---

### AdminGameViewModel
A read projection for admin/testing use.

This is not a core domain entity.
It is a backend-assembled view.

It should include:
- full board state
- all seats
- all channels/messages
- all commitments
- phase controls
- adjudication traces
- submission status by seat

---

## Core relationships

### Reference relationships
- A RulesetDefinition has many MapDefinitions.
- A MapDefinition has many PowerDefinitions.
- A MapDefinition has many ProvinceDefinitions.
- A MapDefinition has many BoardLocationDefinitions.
- A MapDefinition has many AdjacencyDefinitions.
- A MapDefinition has many StartingUnitDefinitions.
- A MapDefinition has many StartingSupplyCenterControlDefinitions.

### Runtime relationships
- A Game references one RulesetDefinition.
- A Game references one MapDefinition.
- A Game has many Seats.
- A Game has many Phases.
- A Game has many PositionSnapshots.
- A Game has many Units.
- A Game has many SupplyCenterControl records.

### Orders relationships
- A Phase has many OrderDrafts.
- A Phase has many OrderSubmissions.
- An OrderSubmission has many OrderEntries.
- A Phase may have one or more AdjudicationRuns.
- An AdjudicationRun has many ResolutionEntries.

### Communication relationships
- A Game has many Channels.
- A Channel has many Messages.
- A direct Channel has exactly two participating Seats.
- A Message belongs to one Phase.
- A CommitmentRecord belongs to one Game.
- A CommitmentRecord originates from one Message.
- A CommitmentRecord may link to additional Messages.
- A CommitmentRecord has many CommitmentParticipants.

### Access relationships
- A User may have access to many Seats across games.
- A Seat may be accessible by one or more Users depending on environment and tooling.
- Admin/testing access is an application-layer concern, not a game-rule concept.

---

## Important non-goals for this model

This model does not yet attempt to finalize:
- the exact commitment-tracking terminology
- the final commitment lifecycle vocabulary
- the final database schema
- the final transport/API format
- the future AI agent internals

Those belong in later docs.

---

## Decisions currently locked

The following decisions are considered established for the platform MVP:

- Seat is the primary gameplay actor boundary.
- Ruleset and map are separate, with map under ruleset.
- Phase is explicit and typed.
- Board locations must support coasts cleanly.
- Units have stable identity across the game.
- Resolved board snapshots should exist per phase.
- Event history should also exist.
- Orders are structured internal objects.
- Messages are phase-linked.
- Commitments are made through messages and linked back to specific messages.
- Read models are separate from core domain entities.

---

## Open questions for later refinement

These are intentionally deferred:
- exact commitment-tracking terminology
- exact board-location representation style
- exact normalized schema vs document payload tradeoffs
- exact event taxonomy
- exact order-entry field shape for every order type
- exact admin/testing access model
