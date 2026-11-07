---
weight: 1
---
# Overview

The Battery System is a core critical component of the CyberLab Core System. It tracks charge
level, publishes local Battery state messages, sends `battery0` telemetry to the Power System
JSON-RPC contract, and operates in realistic dispatch-driven mode by default.

In realistic mode, Battery waits for dispatch requests from the Power System and recharges when
Power reports surplus primary generation through `BatteryRequestedChargeKw`. It no longer runs on
standalone threshold-only demo behaviour unless a feeder input mode is selected.

## Ports

| Service | Host port | Container port | Description |
| --- | ---: | ---: | --- |
| UI Dashboard | `5000` | `80` | nginx-served Battery dashboard |
| Backend API | `5001` | `5001` | Python Battery service and health endpoint |

Default local URLs:

- Dashboard: `http://localhost:5000/`
- Health check: `http://localhost:5001/health`
- Full state API: `http://localhost:5001/api/v1/state`

## Quick Start

Add `battery` to your deployment config, whether that be through a lab config or directly, and deploy your branch.

## Battery States

| State | Description | Power supplied |
| --- | --- | ---: |
| `standby` | Battery is idle and available when charge protection is clear | `0.0 kW` |
| `charging` | Battery is recharging from Power surplus primary generation | `0.0 kW` |
| `discharging` | Power System requested Battery discharge support | Requested/applied kW |
| `empty` | Battery depleted, no discharge support available | `0.0 kW` |

## Input Modes

| Mode | Description |
| --- | --- |
| `realistic` | Default. Reads `power0`, supplies targeted dispatch, and recharges from `BatteryRequestedChargeKw` |
| `standby` | Fixed standby feeder mode |
| `charging` | Fixed charging feeder mode |
| `discharging` | Fixed discharging feeder mode |
| `low_charge` | Simulated low charge state |
| `empty` | Simulated empty battery |

Switch mode through the backend API:

```bash
curl -X POST http://localhost:5001/api/v1/input-mode \
  -H "Content-Type: application/json" \
  -d '{"mode": "realistic"}'
```

The dashboard calls the same API through nginx on `http://localhost:5001/api/v1/input-mode`.

## Power System Integration

Battery communicates with Power System via JSON-RPC on port `5001`, sending `currentData`
telemetry on the configured Power telemetry interval.

Telemetry channels include:

| Channel | Description |
| --- | --- |
| `battery0/Status` | Power contract status, currently `ONLINE` when the Battery service is running |
| `battery0/OperatingState` | Internal state such as `standby`, `charging`, `discharging`, or `empty` |
| `battery0/StateOfChargePct` | Current charge level from `0` to `100` |
| `battery0/AvailableDischargeKw` | Discharge power currently available for support |
| `battery0/AvailableChargeKw` | Charge power currently available |
| `battery0/SupportAvailable` | Whether Battery can be requested for support |
| `battery0/ChargeProtectionActive` | Whether low-charge protection blocks discharge support |
| `battery0/ChargeProtectionMinPct` | Threshold where charge protection starts |
| `battery0/ChargeRecoveryPct` | Threshold where charge protection clears |
| `battery0/Reason` | Human-readable state reason |
| `battery0/Faulted` | Fault state, currently `false` for the prototype |
| `battery0/LastUpdateTs` | ISO-8601 UTC timestamp |

Battery reads the read-only `power0` coordination channels, especially:

- `Mode`
- `DispatchRequestId`
- `BatterySupportRequested`
- `BatteryRequestedChargeKw`
- `BatteryRequestedDischargeKw`
- `BatteryDispatchAckState`

If the active dispatch request targets Battery and the current charge can satisfy the requested
discharge while respecting the reserve threshold, Battery sends `componentJsonApi.ackDispatch` as
`ACCEPTED`, applies discharge, sends fresh telemetry, then acknowledges `APPLIED`. Unsupported
requests are acknowledged as `REJECTED`.

When `BatteryRequestedChargeKw` is greater than `0` and Power is in `NORMAL`, Battery charges at
up to `charge_rate` per configured heartbeat. At or below `charge_protection_min_pct`, Battery
stops offering support until charge reaches `charge_recovery_pct`.

If the Power System is unreachable, the Battery service logs the issue and continues running.

## Configuration

Configuration is managed via `config.yaml`, with deployment values rendered from
`ansible/playbooks/container_projects/battery/templates/config.yaml.j2`.

| Parameter | Default | Description |
| --- | ---: | --- |
| `initial_charge` | `0` | Starting charge level |
| `max_charge` | `100` | Maximum capacity |
| `charge_rate` | `20` | Charge amount per interval |
| `discharge_rate` | `20` | Discharge amount per interval |
| `support_power_kw` | `80` | Maximum discharge power advertised to Power System |
| `charge_protection_min_pct` | `20` | Threshold where Battery stops offering discharge support |
| `charge_recovery_pct` | `60` | Threshold where Battery can offer support again |
| `heartbeat_interval_seconds` | `10` | Internal simulation/message interval |
| `input_mode` | `realistic` | Default input mode |
| `power_integration.enabled` | `true` | Enable Power System integration |
| `power_integration.url` | `http://localhost:9001` | Base Power System URL before environment overrides |
| `power_integration.request_timeout_seconds` | `5` | HTTP request timeout |
| `power_integration.component_id` | `battery0` | Component id used in telemetry |
| `power_integration.support_min_charge_pct` | `20` | Minimum charge before Battery offers support |
| `power_integration.heartbeat_interval_seconds` | `10` | Power System telemetry interval |

Environment variables can override the Power integration endpoint and timing:

- `POWER_INTEGRATION_ENABLED`
- `POWER_JSONRPC_URL`
- `POWER_REQUEST_TIMEOUT`
- `POWER_TELEMETRY_INTERVAL_SECONDS`
- `BATTERY_COMPONENT_ID`
- `BATTERY_SUPPORT_MIN_CHARGE_PCT`

## API Endpoints

| Endpoint | Method | Description |
| --- | --- | --- |
| `/health` | `GET` | Service health and compact Battery state |
| `/api/v1/state` | `GET` | Full Battery state payload |
| `/api/v1/input-mode` | `GET` | Current input mode and available modes |
| `/api/v1/input-mode` | `POST` | Switch input mode |

## Tests

Run Battery power-client tests from the repository root:

```bash
PYTHONPATH=containers/battery/src \
  python3 -m pytest containers/battery/tests/test_power_client.py
```
