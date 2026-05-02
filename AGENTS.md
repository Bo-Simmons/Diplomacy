# AGENTS.md

## Repository purpose

This repository will eventually contain **two closely related systems**:

1. the **Diplomacy platform**
2. the **AI player system**

These systems belong in the same repository, but they are **not** the same subsystem and must not be tightly coupled.

The **current implementation phase** is the **platform MVP**.

That means:
- the platform is the current build target
- the AI player system is a planned future subsystem in this same repository
- current work must preserve clean boundaries for later AI integration
- do not implement the AI player system unless explicitly instructed

---

## Current project phase

The repository is currently in the **planning and scaffolding stage for the platform MVP**.

The immediate goal is to define and implement a local-first Diplomacy platform that is strong enough to serve as the future execution environment for human and AI players.

The immediate goal is **not** to build AI players yet.

Prioritize:
1. architecture clarity
2. durable documentation
3. clean subsystem boundaries
4. local usability
5. correctness over polish

---

## Long-term repository structure

This repository should be organized around the distinction between platform, AI, and shared concerns.

Expected documentation structure:

- `docs/overview/`
- `docs/platform/mvp/`
- `docs/platform/future/`
- `docs/ai/overview/`
- `docs/ai/future/`
- `docs/shared/`
- `docs/decisions/`
- `docs/prompts/`

Use these directories intentionally.

### Directory intent

#### `docs/overview/`
Repository-level orientation documents.

Examples:
- repository roadmap
- architecture overview
- terminology

#### `docs/platform/mvp/`
Authoritative docs for the current platform MVP.

This is the main working area right now.

#### `docs/platform/future/`
Platform concerns that are intentionally deferred until after the MVP.

Examples:
- deployment
- production authentication
- operations
- scaling
- public multiplayer features

#### `docs/ai/overview/`
High-level future-facing AI subsystem docs.

These may define intended boundaries and interfaces, but should not trigger implementation work unless explicitly requested.

#### `docs/ai/future/`
Detailed future AI-player design docs.

Examples:
- seat interface
- context model
- memory model
- negotiation pipeline
- provider/model abstraction

#### `docs/shared/`
Cross-cutting documents used by both platform and AI.

Examples:
- variant data model
- visibility rules
- event log and replay model
- canonical terminology

#### `docs/decisions/`
Architecture decision records.

Use this for decisions that should remain stable and inspectable over time.

#### `docs/prompts/`
Reusable task briefs and templates intended to be given to coding agents.

---

## Core strategic rule

This repo contains both the platform and the future AI player system, but the **current active build target is the platform MVP**.

Do not confuse:
- “this repository eventually contains AI”
with
- “AI implementation work should begin now”

Those are not the same thing.

Until explicitly told otherwise:
- design for future AI integration
- preserve boundaries for future AI integration
- do **not** build the AI player system yet

---

## What the platform MVP is

The platform MVP is a **local-first, server-authoritative Diplomacy platform** with the following requirements:

- It must run fully locally.
- Deployment is not required for current work.
- A single developer must be able to open the same game from multiple seat perspectives on one machine.
- The platform must support:
  - universal press
  - direct communication between exactly two players
  - structured diplomatic artifacts inside the communication system:
    - Agreement
    - Pact
    - Treaty
- The rules are fixed to standard Diplomacy for now.
- Map and power definitions must be designed in a data-driven, variant-ready way from the beginning.

---

## What the future AI subsystem is expected to be

The future AI subsystem is expected to:
- sit on top of the platform rather than replace it
- interact with the platform through clean interfaces
- operate as a seat occupant, not as a special-case game engine
- remain model/provider agnostic
- maintain per-player context and diplomatic memory

This future direction should influence present boundaries, but should not trigger premature implementation.

---

## Hard architectural constraints

These constraints apply unless explicitly overridden by the user.

### 1) Platform and AI must remain separate subsystems

Do not entangle:
- platform game logic
- platform message storage
- adjudication
- UI rendering
- AI reasoning
- AI memory management
- model/provider integration

The platform should remain usable even before AI exists.

The future AI subsystem should be able to plug into the platform through clean interfaces.

---

### 2) Standard rules are fixed for the current platform MVP

Do not invent:
- alternate adjudication rules
- enforced diplomacy promises
- custom house rules
- nonstandard gameplay mechanics

unless explicitly instructed.

The current effort is standard-rules-first.

---

### 3) Variant readiness is required at the data/model layer

Do not hardcode the standard map so deeply that alternate maps or alternate power sets become difficult to support later.

The intended shape is:

- rules implementation is standard-first
- map topology is data-driven
- powers are data-driven
- starting units are data-driven
- province metadata is data-driven
- victory threshold is data-driven

Do not over-generalize the rules engine for hypothetical future variants, but do keep map and setup data first-class.

---

### 4) The adjudicator must remain a clean boundary

Do not mix adjudication logic with:
- UI logic
- message logic
- diplomatic artifact storage
- AI planning logic
- provider/model code

Conceptually, the adjudicator should accept:
- variant/setup data
- current game state
- submitted orders

and return:
- resolved state
- adjudication results
- retreat requirements
- adjustment requirements
- relevant phase outputs

Keep this boundary explicit.

---

### 5) Seat visibility is sacred

Every seat must only see what that seat is allowed to see.

This applies to:
- direct messages
- private diplomatic artifacts
- public diplomatic artifacts
- seat-specific pending actions
- future AI seat context
- dev tools outside explicit debug/admin contexts

Do not take shortcuts that collapse visibility boundaries just because current work is local-only.

---

### 6) Local-first is the default

Do not optimize for:
- deployment
- production cloud infra
- external auth providers
- hosted multiplayer operations

unless explicitly asked.

The platform must be locally runnable and locally testable first.

---

### 7) Structured diplomacy artifacts are communication-layer artifacts

Agreement, Pact, and Treaty are part of the communication layer.

They are:
- socially meaningful
- storable
- queryable
- visible according to visibility rules
- useful for future human and AI reasoning

They are **not** mechanically binding in the current platform MVP.

Do not make the judge enforce them.

---

## Communication model rules

The platform MVP should support:

- **Universal press**
  - visible to all seats

- **Direct press**
  - visible only to the two participating seats

Structured diplomatic artifacts must be layered on top of this communication model.

### Artifact semantics

#### Agreement
A lighter-weight, usually short-horizon arrangement.
Often operational and turn-relevant.

Examples:
- DMZ proposal for a specific province this turn
- temporary support promise
- one-turn coordination plan

#### Pact
A broader private arrangement.
Usually more complex than an Agreement and often multi-clause.

Examples:
- private non-aggression understanding
- region-wide DMZ
- private coordination plan across several moves

#### Treaty
A formal artifact that is published to universal press.

Examples:
- public alliance
- public non-aggression arrangement
- public territorial understanding

### Important rule
Agreement / Pact / Treaty must be implemented as communication-layer artifacts with lifecycle state such as:
- proposed
- acknowledged
- rejected
- countered
- superseded
- expired
- broken

They must not be implemented as hard rules that force player orders.

---

## Documentation-first workflow

Documentation is part of the implementation in this repository.

Before making major structural changes, read the relevant docs first.

Expected documentation pathing:

### Platform MVP docs
- `docs/platform/mvp/01-product-scope.md`
- `docs/platform/mvp/02-non-goals.md`
- `docs/platform/mvp/03-domain-model.md`
- `docs/platform/mvp/04-phase-state-machine.md`
- `docs/platform/mvp/05-variant-schema.md`
- `docs/platform/mvp/06-press-and-diplomacy-artifacts.md`
- `docs/platform/mvp/07-local-dev-and-seat-testing.md`
- `docs/platform/mvp/08-adjudication-boundary.md`
- `docs/platform/mvp/09-api-surface.md`
- `docs/platform/mvp/10-ui-surfaces.md`
- `docs/platform/mvp/11-implementation-plan.md`

### Shared docs
Examples:
- `docs/shared/variants-and-data-model.md`
- `docs/shared/visibility-and-permissions.md`
- `docs/shared/event-log-and-replay.md`

### AI docs
Examples:
- `docs/ai/overview/ai-system-scope.md`
- `docs/ai/overview/platform-ai-boundary.md`

If a task depends on a missing doc:
- do not silently invent large architecture
- either keep the implementation tightly scoped
- or create/propose the minimal necessary doc first

---

## How to classify work

When given a task, first classify it as one of:

- repository overview / orientation
- platform MVP documentation
- platform MVP implementation
- platform future planning
- AI future planning
- shared schema / contract design
- architecture decision recording
- coding-agent prompt preparation

Use this classification to decide where docs and code belong.

---

## How to work in this repo

When assigned a task:

1. Determine which subsystem it belongs to:
   - platform
   - AI
   - shared
   - overview
   - decision log
   - prompting/instructions

2. Read the relevant docs first.

3. Preserve these boundaries:
   - platform vs AI subsystem
   - adjudication vs messaging
   - public communication vs private communication
   - seat-visible data vs debug/admin visibility
   - variant data vs rules logic
   - durable docs vs one-off implementation convenience

4. Prefer incremental, reviewable changes.

5. If a change affects architecture or long-term assumptions, update the relevant docs.

6. If a task is under-specified, make the smallest reasonable assumption and state it.

---

## What not to do unless explicitly asked

Do not proactively add:
- AI player implementations
- model/provider integrations
- prompt systems for agents
- vector memory systems
- negotiation engines
- deployment pipelines
- production auth
- matchmaking
- rankings
- tournament systems
- spectator mode
- notifications
- mobile optimization
- broad nonstandard variant rules
- broad refactors unrelated to the requested task

Do not overbuild.

---

## Planning expectations

For complex work:
- plan first
- keep scope narrow
- preserve future extensibility where it is clearly required
- do not build speculative systems just because they may be useful later

If a task spans multiple architectural areas, prefer:
- a short written plan
- then staged execution

If current work must reserve space for the future AI subsystem, do so through:
- interfaces
- clean data contracts
- docs
- boundaries

not through premature implementation.

---

## Definition of done

A task is only done when all relevant conditions are satisfied.

### Always required
- it respects the current project phase
- it respects the platform / AI subsystem boundary
- it does not add unrequested scope
- it is understandable to a future coding agent reading the repo
- any changed architectural assumptions are reflected in docs

### If code is involved
- the change is locally runnable
- local workflows remain simple
- tests or validation are updated where appropriate
- visibility rules are preserved
- standard-rules assumptions are not accidentally broken

### If docs are involved
- they are placed in the correct directory
- they do not misclassify platform work as AI work or vice versa
- they are durable enough to guide future agent work

---

## Preferred style of changes

Prefer:
- explicit structure
- strong naming
- small composable modules
- data-driven definitions where required
- comments when architectural intent could otherwise be lost
- repository organization that reflects long-term subsystem boundaries

Avoid:
- hidden coupling
- magic behavior
- over-abstraction for hypothetical futures
- platform code that assumes AI internals
- AI planning docs that quietly dictate platform implementation without explicit agreement

---

## Decision records

Use `docs/decisions/` for durable architectural decisions that should remain visible over time.

Examples of appropriate decision topics:
- the repo contains both platform and AI subsystems
- the platform MVP comes before AI implementation
- variants are data-driven from the start
- diplomacy artifacts are communication-layer constructs, not judge mechanics

Decision records should be short, stable, and explicit about consequences.

---

## Prompts for coding agents

Reusable task prompts should live under `docs/prompts/`.

These prompts should:
- reference relevant docs
- state scope clearly
- define non-goals
- define what counts as done
- avoid asking the coding agent to invent broad architecture from scratch

Prompt files should not replace durable repo docs.

---

## If instructions conflict

Priority order:

1. direct user request
2. this `AGENTS.md`
3. relevant documents in:
   - `docs/platform/`
   - `docs/ai/`
   - `docs/shared/`
   - `docs/overview/`
   - `docs/decisions/`
4. local implementation convenience

If something is unclear, do not improvise aggressively.
Make the smallest reasonable assumption, state it, and preserve future flexibility where it is already part of the known plan.
