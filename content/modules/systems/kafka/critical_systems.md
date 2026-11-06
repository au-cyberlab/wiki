---
title: Critical Systems
---

# Critical System Kafka Integration

This wiki page is the canonical location for the CyberLab critical system Kafka
integration contract.

## Goal

This document defines a lightweight Kafka message standard for the CyberLab
core systems so that the `power`, `battery`, `water`, and `gas` systems can be
developed in parallel without blocking early implementation.

The design goal is to make the integration contract stable enough for the team
to build against now, while still allowing additive changes later.

## Scope

The systems in scope are:

- `power`
- `battery`
- `water`
- `gas`

The required behaviour is:

- `power` outputs power updates to Kafka for the rest of the CyberLab
- `water` generates power for `power`
- `gas` generates power for `power`
- `battery` supplies backup power when generation is insufficient or when
  `power` is unavailable
- if `water` or `gas` go down, less power is generated and some power is drawn
  from `battery`
- if `power` goes down, `water` and `gas` automatically shut down
- if `power` goes down, `battery` becomes the only source of power until it is
  empty

## Design Principles

This standard is intentionally lightweight so it does not slow down early-stage
development.

- All messages are JSON
- All messages use the same envelope
- Payloads are message-type specific
- New optional fields may be added later without breaking old consumers
- Existing required fields must not be renamed or removed within `1.x`
- Unknown fields must be ignored by consumers
- No Schema Registry is required for the first implementation

## Assumptions

- `power` acts as the main coordinator and publishes the official power output
  seen by the rest of the CyberLab
- `water` and `gas` are generation sources, not final output systems
- `battery` may be in `standby`, `charging`, `discharging`, or `empty`
- `battery` charging is modelled as energy available from the generation side
  through the `power` system
- if a separate `power plant` component is introduced later, it should extend
  this contract by adding new `source` values rather than renaming existing
  fields

## Topics

The project should use four Kafka topics.

| Topic | Purpose |
| --- | --- |
| `cyberlab.core.heartbeat` | Liveness of each core system |
| `cyberlab.core.state` | State, generation, and battery updates |
| `cyberlab.core.command` | Optional control and test commands |
| `cyberlab.power.output` | Official power output for the rest of CyberLab |

## Standard Message Envelope

Every Kafka message must use this envelope.

```json
{
  "schema_version": "1.0",
  "message_type": "generation.update",
  "source": "water",
  "target": "power",
  "timestamp": "2026-03-22T10:00:00Z",
  "event_id": "8db0d0b3-542f-4f75-936e-bdd0f685f1dd",
  "correlation_id": null,
  "payload": {}
}
```

### Required Fields

| Field | Type | Description |
| --- | --- | --- |
| `schema_version` | `string` | Message contract version. Start with `1.0` |
| `message_type` | `string` | Logical message type |
| `source` | `string` | Producing system |
| `target` | `string \| null` | Intended consumer, `all`, or `null` |
| `timestamp` | `string` | ISO 8601 UTC timestamp |
| `event_id` | `string` | Unique event ID, ideally UUID |
| `correlation_id` | `string \| null` | Used to correlate related events |
| `payload` | `object` | Message-specific data |

### Allowed `source` Values

- `power`
- `battery`
- `water`
- `gas`
- `integration-test`
- `orchestrator`

### Allowed `target` Values

- `power`
- `battery`
- `water`
- `gas`
- `all`
- `null`

## Message Types

The first version of the project should standardise six message types.

| Message Type | Topic | Purpose |
| --- | --- | --- |
| `heartbeat` | `cyberlab.core.heartbeat` | Liveness signal |
| `component.state` | `cyberlab.core.state` | Generic component status |
| `generation.update` | `cyberlab.core.state` | Generation report from `water` or `gas` |
| `battery.update` | `cyberlab.core.state` | Battery charge and supply report |
| `control.command` | `cyberlab.core.command` | Optional external command or fault injection |
| `power.output` | `cyberlab.power.output` | Official power output for other systems |

## Message Definitions

### 1. `heartbeat`

Topic: `cyberlab.core.heartbeat`

Required payload fields:

| Field | Type | Description |
| --- | --- | --- |
| `status` | `string` | Usually `online`, `degraded`, or `offline` |
| `sequence` | `integer` | Monotonic counter for debugging |

Example:

```json
{
  "schema_version": "1.0",
  "message_type": "heartbeat",
  "source": "power",
  "target": "all",
  "timestamp": "2026-03-22T10:00:00Z",
  "event_id": "d667e2bd-792f-43c7-ad8c-7f8e7f49601f",
  "correlation_id": null,
  "payload": {
    "status": "online",
    "sequence": 120
  }
}
```

### 2. `component.state`

Topic: `cyberlab.core.state`

Required payload fields:

| Field | Type | Description |
| --- | --- | --- |
| `status` | `string` | Current component state |
| `reason` | `string \| null` | Optional explanation |

Suggested `status` values:

- `online`
- `degraded`
- `offline`
- `shutdown`
- `standby`
- `charging`
- `discharging`
- `empty`
- `fault`

Example:

```json
{
  "schema_version": "1.0",
  "message_type": "component.state",
  "source": "gas",
  "target": "power",
  "timestamp": "2026-03-22T10:01:00Z",
  "event_id": "5fc98d0c-054c-4cdd-9d95-db0a45598d64",
  "correlation_id": null,
  "payload": {
    "status": "offline",
    "reason": "manual fault injection"
  }
}
```

### 3. `generation.update`

Topic: `cyberlab.core.state`

Produced by:

- `water`
- `gas`

Required payload fields:

| Field | Type | Description |
| --- | --- | --- |
| `status` | `string` | Current state of the generator |
| `power_generated` | `number` | Current output produced by the system |
| `unit` | `string` | Use `kW` for the first version |

Optional payload fields:

| Field | Type | Description |
| --- | --- | --- |
| `max_capacity` | `number` | Maximum output capacity |
| `efficiency` | `number` | Optional ratio from `0.0` to `1.0` |

Example:

```json
{
  "schema_version": "1.0",
  "message_type": "generation.update",
  "source": "water",
  "target": "power",
  "timestamp": "2026-03-22T10:02:00Z",
  "event_id": "ed7c6751-b2f9-4969-947d-6d2f30fbd422",
  "correlation_id": null,
  "payload": {
    "status": "online",
    "power_generated": 45.0,
    "unit": "kW",
    "max_capacity": 50.0
  }
}
```

### 4. `battery.update`

Topic: `cyberlab.core.state`

Produced by:

- `battery`

Required payload fields:

| Field | Type | Description |
| --- | --- | --- |
| `status` | `string` | `standby`, `charging`, `discharging`, or `empty` |
| `charge_level` | `number` | Remaining battery charge as percentage `0-100` |
| `power_supplied` | `number` | Current battery output supplied to the system |
| `unit` | `string` | Use `kW` for the first version |

Optional payload fields:

| Field | Type | Description |
| --- | --- | --- |
| `charge_rate` | `number` | Positive while charging |
| `discharge_rate` | `number` | Positive while discharging |

Example:

```json
{
  "schema_version": "1.0",
  "message_type": "battery.update",
  "source": "battery",
  "target": "power",
  "timestamp": "2026-03-22T10:03:00Z",
  "event_id": "3459e978-bd96-43b6-b0aa-e8bf31eb9641",
  "correlation_id": null,
  "payload": {
    "status": "discharging",
    "charge_level": 82.5,
    "power_supplied": 10.0,
    "unit": "kW",
    "discharge_rate": 10.0
  }
}
```

### 5. `control.command`

Topic: `cyberlab.core.command`

This message type is optional but strongly recommended because it makes testing,
fault injection, and demos easier.

Required payload fields:

| Field | Type | Description |
| --- | --- | --- |
| `action` | `string` | Command action |
| `value` | `object \| string \| number \| null` | Command input |
| `reason` | `string \| null` | Optional explanation |

Suggested actions:

- `start`
- `stop`
- `shutdown`
- `set_output`
- `set_charge`
- `inject_fault`
- `recover`

Example:

```json
{
  "schema_version": "1.0",
  "message_type": "control.command",
  "source": "integration-test",
  "target": "water",
  "timestamp": "2026-03-22T10:04:00Z",
  "event_id": "7aa35fd2-3fab-4b4e-9eb7-f0e4e53fb161",
  "correlation_id": null,
  "payload": {
    "action": "inject_fault",
    "value": {
      "status": "offline"
    },
    "reason": "test partial generation loss"
  }
}
```

### 6. `power.output`

Topic: `cyberlab.power.output`

Produced by:

- `power`

This is the official power event that downstream CyberLab systems should read.

Required payload fields:

| Field | Type | Description |
| --- | --- | --- |
| `status` | `string` | Overall power system status |
| `total_power_available` | `number` | Total power currently available |
| `battery_power_used` | `number` | Power being supplied by the battery |
| `unit` | `string` | Use `kW` for the first version |
| `source_breakdown` | `object` | Per-source output values |

Suggested `source_breakdown` structure:

```json
{
  "water": 45.0,
  "gas": 30.0,
  "battery": 10.0
}
```

Optional payload fields:

| Field | Type | Description |
| --- | --- | --- |
| `demand` | `number` | Optional demand level if modelled |
| `battery_charge_level` | `number` | Battery percentage for convenience |
| `unserved_power` | `number` | Shortfall if battery is empty or insufficient |

Example:

```json
{
  "schema_version": "1.0",
  "message_type": "power.output",
  "source": "power",
  "target": "all",
  "timestamp": "2026-03-22T10:05:00Z",
  "event_id": "73242f6f-213f-4425-96ec-ff6a276ba294",
  "correlation_id": null,
  "payload": {
    "status": "degraded",
    "total_power_available": 85.0,
    "battery_power_used": 10.0,
    "unit": "kW",
    "source_breakdown": {
      "water": 45.0,
      "gas": 30.0,
      "battery": 10.0
    },
    "battery_charge_level": 82.5,
    "unserved_power": 0.0
  }
}
```

## System Responsibilities

### `power`

Publishes:

- `heartbeat`
- `component.state`
- `power.output`

Subscribes to:

- `generation.update` from `water`
- `generation.update` from `gas`
- `battery.update` from `battery`
- `component.state` from all systems
- `control.command` targeted to `power` or `all`

Responsibilities:

- maintain the official view of current power output
- aggregate generation from `water` and `gas`
- decide when battery contribution is required
- publish the official `power.output` events

### `water`

Publishes:

- `heartbeat`
- `component.state`
- `generation.update`

Subscribes to:

- `heartbeat` from `power`
- `control.command` targeted to `water` or `all`

Responsibilities:

- report its current generation
- stop automatically if `power` is unavailable

### `gas`

Publishes:

- `heartbeat`
- `component.state`
- `generation.update`

Subscribes to:

- `heartbeat` from `power`
- `control.command` targeted to `gas` or `all`

Responsibilities:

- report its current generation
- stop automatically if `power` is unavailable

### `battery`

Publishes:

- `heartbeat`
- `component.state`
- `battery.update`

Subscribes to:

- `heartbeat` from `power`
- `control.command` targeted to `battery` or `all`

Responsibilities:

- stay in standby while generation is sufficient
- provide power when generation is insufficient
- provide all power when `power` is unavailable
- continue until empty

## Failure and Recovery Rules

### Rule 1: Water or Gas Failure

If `water` or `gas` go down:

- the failed system publishes `component.state`
- generation from that source drops to `0`
- `power` recalculates total generation
- `battery` supplies any required shortfall if charge is available
- `power` publishes a degraded `power.output`

### Rule 2: Power Failure

If `power` goes down:

- `power` heartbeats stop
- `water` detects missing `power` heartbeat and shuts down
- `gas` detects missing `power` heartbeat and shuts down
- `battery` detects missing `power` heartbeat and switches to emergency
  discharge mode
- `battery` continues supplying power until charge is `0`

### Rule 3: Battery Empty

If `battery` reaches `0` charge:

- `battery` publishes `battery.update` with `status: "empty"`
- `power.output` should report `battery_power_used: 0`
- if demand is modelled, `unserved_power` should be greater than `0`

### Rule 4: Recovery

If a failed component recovers:

- it resumes publishing `heartbeat`
- it publishes `component.state` with a healthy status
- it resumes sending normal state updates
- `power` resumes aggregation using that source

## Heartbeat Rules

Recommended initial heartbeat policy:

- each active system publishes a `heartbeat` every `2` seconds
- a component should consider another component unavailable after `3` missed
  heartbeats
- this gives a timeout window of approximately `6` seconds

This policy is simple enough for the first version and can be tuned later.

## Kafka Key Recommendation

To keep ordering predictable per component, use the following keys.

| Topic | Recommended Kafka Key |
| --- | --- |
| `cyberlab.core.heartbeat` | `source` |
| `cyberlab.core.state` | `source` |
| `cyberlab.core.command` | `target` |
| `cyberlab.power.output` | `power` |

## Compatibility Rules

These rules are important because the standard must not slow down early-stage
development.

- consumers must ignore unknown fields
- producers may add optional fields
- required fields in this document must remain stable within version `1.x`
- message meaning should not change without bumping `schema_version`
- if a payload is missing a required field, consumers should log the error and
  safely ignore that message

## Team Implementation Contract

Each teammate should only rely on the following minimum contract.

### Required for All Systems

- publish `heartbeat`
- publish `component.state`
- use the standard envelope
- use `schema_version: "1.0"`
- serialize messages as JSON

### Required for `water` and `gas`

- publish `generation.update`
- shut down if `power` heartbeat is missing

### Required for `battery`

- publish `battery.update`
- enter discharge mode when generation is insufficient
- enter emergency discharge mode if `power` heartbeat is missing

### Required for `power`

- subscribe to generation and battery updates
- publish official `power.output`
- publish its own `heartbeat`

## Suggested Validation Approach

For the first implementation:

- define one Pydantic model for the common envelope
- define one Pydantic model per payload type
- validate only required fields at runtime
- log and drop invalid messages instead of crashing the service

This keeps validation useful without becoming a development bottleneck.

## Short Version for Team Chat

Use this if you want to send a short implementation note to teammates.

```text
We will use a lightweight Kafka contract for the CyberLab core systems.

Topics:
- cyberlab.core.heartbeat
- cyberlab.core.state
- cyberlab.core.command
- cyberlab.power.output

Every message must include:
- schema_version
- message_type
- source
- target
- timestamp
- event_id
- correlation_id
- payload

Rules:
- use JSON only
- ignore unknown fields
- do not rename/remove required fields in v1.x
- water and gas publish generation.update
- battery publishes battery.update
- power publishes the official power.output
- if power heartbeat disappears, water and gas shut down and battery becomes
  the only power source until empty
```

## Next Step

Once the team agrees on this document, the next implementation step should be a
small shared Python package containing:

- common Kafka config
- shared Pydantic message models
- serializer and deserializer helpers
- topic name constants

That shared package can then be imported by all four systems to keep the actual
integration code consistent.
