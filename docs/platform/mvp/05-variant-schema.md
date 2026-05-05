# Variant Schema

## Purpose

This document defines the conceptual authored schema for ruleset and map content in the platform MVP.

It exists to:
- prevent hardcoding the standard board into platform logic
- keep the current implementation concrete around the standard ruleset and standard map
- preserve a clear path for adding more maps under a ruleset later
- define enough schema-level structure to guide implementation without choosing a final file format or database schema

This is an architecture/schema document. It is not a JSON/YAML specification, database schema, API contract, or tooling plan.

---

## Ruleset To Map Structure

Rulesets own maps.

A **ruleset** defines gameplay semantics, such as supported phase kinds, supported unit types, supported order types, and adjudication semantics.

A **map** exists under one ruleset and defines board data, such as powers, provinces, board locations, adjacencies, home centers, starting units, starting supply-center control, victory threshold, and rendering metadata.

For the platform MVP:
- only the standard ruleset is implemented
- only the standard map is shipped
- the schema should still make adding another map under the standard ruleset straightforward later

The platform should not treat the standard map as hardcoded platform behavior.

---

## Core Reference Entities

### `RulesetDefinition`

Defines gameplay semantics shared by maps under the ruleset.

For the MVP, this is the standard Diplomacy ruleset. It owns the standard-ruleset phase cycle described in `docs/platform/mvp/04-phase-state-machine.md`.

### `MapDefinition`

Defines a playable map under a ruleset.

Map-level data includes:
- display metadata
- power count
- victory threshold
- references to powers, provinces, locations, adjacencies, home centers, starting units, starting control, and map features

The victory threshold is map-level configuration because it can vary by map even under the same ruleset.

### `PowerDefinition`

Defines a power that can occupy a seat on a map.

Powers are map-defined, not globally hardcoded.

### `ProvinceDefinition`

Defines a playable province on a map.

Province means playable province only. Inaccessible named regions are map features, not provinces.

### `BoardLocationDefinition`

Defines an explicit occupiable or targetable board location inside one playable province.

Units occupy board locations, not bare provinces.

### `LandAdjacencyDefinition`

Defines the stored movement graph for army movement between land board locations.

### `FleetAdjacencyDefinition`

Defines the stored movement graph for fleet movement between coast and sea board locations.

### `MapFeatureDefinition`

Defines a named, non-playable feature on a map, such as Switzerland or Corsica on the standard map.

Map features support rendering and labeling. They are not provinces.

### `HomeCenterDefinition`

Defines a home-center relation between one power and one supply-center province.

Home-center status is separate from the province's supply-center flag and separate from starting control.

### `StartingUnitDefinition`

Defines a unit present when a game starts on a map.

Starting units reference powers and board locations.

### `StartingSupplyCenterControlDefinition`

Defines which power controls a supply center at game start.

Starting control is separate from home-center status.

---

## Province Schema

A province is a playable map space.

Conceptual fields:
- `province_id`
- `map_id`
- `display_name`
- `abbreviation`
- `province_type`
- `is_supply_center`
- `render_metadata`

Province ids are stable canonical ids and are separate from display names.

Province ids should be human-readable and should encode coarse land/sea category:
- `l_*` for inland or coastal provinces
- `s_*` for sea provinces

Examples:
- `l_spain` may display as `Spain`
- `s_mid_atlantic_ocean` may display as `Mid-Atlantic Ocean`

`province_type` remains explicit and is one of:
- `inland`
- `coastal`
- `sea`

The id prefix is a readability convention. It does not replace the explicit province type field.

Supply-center status is a province/map property represented by `is_supply_center`.

Home-center status is not a province property. It is represented separately by `HomeCenterDefinition`.

Starting supply-center control is not a province property. It is represented separately by `StartingSupplyCenterControlDefinition`.

---

## Board Location Schema

Board locations are explicit and separate from provinces.

Conceptual fields:
- `location_id`
- `map_id`
- `province_id`
- `location_type`
- `display_name_optional`
- `coast_index_optional`
- `unit_type_restrictions`
- `render_metadata`

`location_type` is one of:
- `land`
- `coast`
- `sea`

Location id conventions:
- every non-sea province has exactly one land board location with `_l`
- every coastal province has one or more indexed coast board locations using `_c1`, `_c2`, `_c3`, and so on
- every sea province has exactly one sea board location with `_s`

Examples:
- province: `l_spain`
- Spain land location: `l_spain_l`
- Spain first coast location: `l_spain_c1`
- Spain second coast location: `l_spain_c2`
- province: `s_mid_atlantic_ocean`
- Mid-Atlantic Ocean sea location: `s_mid_atlantic_ocean_s`

Coast direction labels such as `North Coast` or `South Coast` are display metadata. They are not canonical ids.

Board locations within the same province are related by province membership, not by normal movement adjacency.

All board locations in the same playable province share one conceptual occupancy slot. A province may contain at most one occupying unit at a time, even if it has multiple board locations. For example, an army at `l_spain_l` and a fleet at `l_spain_c1` cannot coexist.

---

## Adjacency Schema

Movement adjacency is stored as separate graphs for armies and fleets.

### Land Adjacency

`LandAdjacencyDefinition` defines army movement adjacency.

Conceptual fields:
- `map_id`
- `from_location_id`
- `to_location_id`

Land adjacency should reference only land board locations.

### Fleet Adjacency

`FleetAdjacencyDefinition` defines fleet movement adjacency.

Conceptual fields:
- `map_id`
- `from_location_id`
- `to_location_id`

Fleet adjacency should reference only coast or sea board locations.

### Directionality

Adjacency should be represented as explicit directional rows, even when authored symmetrically.

For symmetric movement, both directions should be represented:
- `from_location_id = a`, `to_location_id = b`
- `from_location_id = b`, `to_location_id = a`

This keeps the schema simple and avoids hidden assumptions if a future map needs asymmetric handling.

### Convoy Reachability

Convoy reachability is derived, not stored as a third adjacency graph.

It should be derived from:
- province-to-location membership
- coastal board locations
- fleet adjacency
- runtime convoying fleets

The map schema should not author convoy reachability as its own graph.

---

## Starting State Schema

Starting state is map data under a ruleset.

### Powers

`PowerDefinition` defines the powers available on the map.

Power metadata may include display name, abbreviation, color, and display order.

### Starting Units

`StartingUnitDefinition` defines units present at game start.

Conceptual fields:
- `map_id`
- `power_id`
- `unit_type`
- `location_id`

Starting units reference board locations.

### Starting Supply-Center Control

`StartingSupplyCenterControlDefinition` defines which power controls each controlled supply center at game start.

Conceptual fields:
- `map_id`
- `province_id`
- `power_id`

Starting control references supply-center provinces.

### Victory Threshold

The victory threshold belongs on `MapDefinition` or equivalent map-level configuration.

For the standard map, this is the supply-center count required to win under the standard ruleset.

---

## Home Centers

Supply-center status, home-center status, and starting control are separate concepts.

### Supply Center

A supply center is a province property.

It is represented by `ProvinceDefinition.is_supply_center`.

### Home Center

A home center is a relation between a power and a supply-center province.

It is represented by `HomeCenterDefinition`.

Conceptual fields:
- `map_id`
- `power_id`
- `province_id`

Home-center status should reference only provinces where `is_supply_center` is true.

### Starting Control

Starting supply-center control is the power that controls a supply center when a game begins.

It is represented by `StartingSupplyCenterControlDefinition`.

A province can be a home center for one power and have starting control assigned through the separate starting-control relation. Keeping these separate avoids conflating map identity, setup state, and runtime control.

---

## Map Features

`MapFeatureDefinition` represents named non-playable map data.

Examples:
- Switzerland
- Corsica
- other labels or regions needed for map rendering and comprehension

Map features:
- are part of map data
- are not provinces
- do not generate board locations
- do not contain units
- do not have supply-center status
- do not participate in land or fleet adjacency
- do not participate in starting control

They exist for rendering, labeling, and map comprehension.

---

## Validation And Invariants

Important schema invariants:

- Each `MapDefinition` belongs to exactly one `RulesetDefinition`.
- Each `PowerDefinition` belongs to exactly one `MapDefinition`.
- Each `ProvinceDefinition` belongs to exactly one `MapDefinition`.
- Each `BoardLocationDefinition` belongs to exactly one playable province.
- Province ids are stable canonical ids and are separate from display names.
- Province ids use `l_*` for inland/coastal provinces and `s_*` for sea provinces.
- `ProvinceDefinition.province_type` remains explicit.
- Every non-sea province has exactly one land board location.
- Every coastal province has one or more coast board locations.
- Coast indexes are unique within a province.
- Every sea province has exactly one sea board location.
- Every starting unit references a valid board location.
- Units occupy board locations, not bare provinces.
- All board locations of a playable province conceptually share one occupancy slot.
- Land adjacency references only land board locations.
- Fleet adjacency references only coast or sea board locations.
- Board locations in the same province are related by membership, not by normal movement adjacency.
- Convoy reachability is derived and is not stored as a third graph.
- Map features are not provinces.
- Map features do not generate board locations.
- Map features do not participate in adjacency.
- Home centers reference supply-center provinces.
- Starting supply-center control references supply-center provinces.
- Home-center status and starting control are separate relations.

---

## Scope Boundary

This document defines conceptual authored schema for the platform MVP.

It does not define:
- final database schema
- final file format
- JSON or YAML layout
- tooling choices
- API payloads
- AI subsystem design
- arbitrary non-Diplomacy mechanics

The MVP should implement the standard ruleset and standard map using this schema shape, while preserving the ruleset/map boundary for future extension.
