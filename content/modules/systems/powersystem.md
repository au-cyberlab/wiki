---
title: Power System
---

# Overview

The Power System is a standalone coordination layer for the CyberLab critical
infrastructure prototype. It receives telemetry from the Water, Gas, and Battery
systems, calculates the current operating mode, and publishes the official
`power0` coordination view for the rest of the environment.

The Power System does not implement the internal behaviour of Water, Gas, or
Battery. Those systems remain owned by their own teams. Power only coordinates
their reported capacity and publishes advisory dispatch requests.

## 1. Setup

The current AWS deployment exposes the Power System service on port `9001`.
The expected Power System host alias is:

| Host | Purpose |
| --- | --- |
| `power_system-server.power_system.lab` | Main Power System service target |

The dashboard and health endpoints are served by this Power System service.

## 2. Usage

### 2.1 Dashboard access

SSH into your cyber range's local jumpbox, then confirm the Power System host is
available:

```bash
dig +short power_system-server.power_system.lab
```

If DNS is not available in the range yet, check the generated host list on the
jumpbox:

```bash
cat ~/hosts.txt
```

On your development machine, forward the Power System dashboard port through the
jumpbox. Replace `USER@JUMPBOX_IP` with your jumpbox login details, and replace
`POWER_SYSTEM_PRIVATE_IP` with the Power System server IP shown in the downloaded
`NetworkMap.png`.

```bash
ssh -i cyberlab-jumpbox-ssh-key.pem \
  -L 9001:POWER_SYSTEM_PRIVATE_IP:9001 \
  -Nf USER@JUMPBOX_IP
```

For example, if the NetworkMap shows `power_system-server.power_system.lab` at
`10.0.6.21`, use:

```bash
ssh -i cyberlab-jumpbox-ssh-key.pem \
  -L 9001:10.0.6.21:9001 \
  -Nf USER@JUMPBOX_IP
```

The `.lab` hostname may resolve correctly after SSHing into the jumpbox, but it
is not always accepted as the target in local SSH port-forward rules. If a
hostname-based tunnel such as `-L 9001:power_system-server.power_system.lab:9001` fails
with `channel open failed: administratively prohibited`, use the private IP from
the NetworkMap instead.

You should now be able to access the dashboard at:

<http://localhost:9001/dashboard>

### 2.2 Health check

After forwarding the service port, check the local health endpoint:

```bash
curl http://localhost:9001/api/v1/health
```

A healthy service should return a JSON response with overall service status and
observability counters.

## 3. Dashboard

The dashboard is designed for operator-style inspection during demos and
integration testing.

It shows:

- current Water, Gas, Battery, Power, and Feeder status cards
- current `power0` operating mode
- dispatch request values for Water, Gas, Battery, and load shedding
- active alerts and acknowledgement state
- system runtime logs
- communication event logs
- recent timedata and history trends

The dashboard follows the same broad shape as OpenEMS Live View and History
View: live values, alerts, logs, and trends are visible as related but separate
surfaces.

## 4. API Quickstart

The Power System exposes REST endpoints for quick testing and an OpenEMS-style
JSON-RPC endpoint for cross-system integration.

Common checks:

```bash
# Service health
curl http://localhost:9001/api/v1/health

# Full component/channel contract
curl http://localhost:9001/api/v1/contract

# Current official Power System output
curl http://localhost:9001/api/v1/components/power0

# Current power0 channels only
curl http://localhost:9001/api/v1/components/power0/channels
```

For manual smoke tests, subsystem telemetry can also be posted through REST:

```bash
curl -X POST http://localhost:9001/api/v1/inputs/water0 \
  -H "Content-Type: application/json" \
  -d '{
    "channels": {
      "Status": "ONLINE",
      "AvailablePowerKw": 60,
      "Faulted": false
    }
  }'
```

The preferred cross-team path is JSON-RPC through:

```text
POST /jsonrpc
```

## 5. Cross-System Protocol Summary

The full contract is maintained in the repository docs. This wiki section is
only the short version for Water, Gas, and Battery integration.

Component ids:

| Component | Owner | Role |
| --- | --- | --- |
| `water0` | Water team | Reports water generation telemetry |
| `gas0` | Gas team | Reports gas generation telemetry |
| `battery0` | Battery team | Reports battery availability and support state |
| `power0` | Power team | Read-only official coordination output |

The preferred telemetry update method is `currentData`:

```json
{
  "jsonrpc": "2.0",
  "id": "water-update-0001",
  "method": "currentData",
  "params": {
    "water0/Status": "ONLINE",
    "water0/AvailablePowerKw": 60,
    "water0/Faulted": false,
    "water0/LastUpdateTs": "2026-05-10T04:00:00Z"
  }
}
```

Subsystems may acknowledge active dispatch requests with
`componentJsonApi.ackDispatch`:

```json
{
  "jsonrpc": "2.0",
  "id": "water-ack-0001",
  "method": "componentJsonApi",
  "params": {
    "componentId": "water0",
    "payload": {
      "method": "ackDispatch",
      "params": {
        "dispatchRequestId": "dispatch-20260510T040003Z-0001",
        "status": "ACCEPTED"
      }
    }
  }
}
```

Power publishes dispatch guidance through read-only `power0` channels. Important
channels include:

| Channel | Meaning |
| --- | --- |
| `Mode` | Current Power System mode |
| `DispatchMode` | Current dispatch decision type |
| `SurplusPrimaryPowerKw` | Primary Water + Gas power remaining after required load is met |
| `WaterRequestedPowerKw` | Requested Water contribution |
| `GasRequestedPowerKw` | Requested Gas contribution |
| `BatteryRequestedChargeKw` | Requested Battery charging from surplus primary power |
| `BatteryRequestedDischargeKw` | Requested Battery discharge |
| `LoadShedRequestedKw` | Requested load shedding |
| `DispatchRequestId` | Current dispatch request identifier |
| `DispatchAckState` | Overall acknowledgement state |

Water, Gas, and Battery should treat dispatch requests as advisory in this
prototype. Hard command execution is not part of the current contract.

## 6. Operating Modes

| Mode | Meaning |
| --- | --- |
| `NORMAL` | Water and Gas primary supply can meet the required load |
| `BATTERY_SUPPORTED` | Primary supply is short, but Battery can cover the gap |
| `DEGRADED` | Some power remains, but total supply cannot meet required load |
| `OUTAGE` | No usable power remains, or all relevant inputs are unavailable |

The coordinator uses the configured required load, latest subsystem telemetry,
fault flags, and input staleness to decide the current mode.

## 7. Diagnostics

### 7.1 Dashboard does not load

Check that the local port forward is still running and that the service health
endpoint responds:

```bash
curl http://localhost:9001/api/v1/health
```

If the health endpoint fails, SSH to the target host and inspect the service:

```bash
systemctl status power-system
```

### 7.2 Telemetry is not updating

Check the current component state:

```bash
curl http://localhost:9001/api/v1/components/water0
curl http://localhost:9001/api/v1/components/gas0
curl http://localhost:9001/api/v1/components/battery0
```

Look for stale `LastUpdateTs`, `Faulted: true`, or a non-`ONLINE` subsystem
status.

### 7.3 Dispatch acknowledgement is missing

Check the current `power0` dispatch request id:

```bash
curl http://localhost:9001/api/v1/components/power0
```

Then confirm the relevant subsystem has sent `componentJsonApi.ackDispatch`
with the current `DispatchRequestId`.

### 7.4 History or logs are empty

Check the observability endpoints:

```bash
curl http://localhost:9001/api/v1/exchange-log
curl http://localhost:9001/api/v1/system-log
curl http://localhost:9001/api/v1/history
```

If persistent history is expected, confirm that the deployment rendered a
`history_storage_path` in the Power System config.

## 8. Known Limits

- Dispatch requests are advisory and require subsystem acknowledgement, but do
  not enforce hard command execution.
- Authentication and transport security are not part of the current prototype.
- The SQLite historian is a lightweight local store, not a production historian.

## 9. Further Reading

- [Power System README][1]
- [Power System design overview][2]
- [Core system JSON protocol][3]
- [Dashboard and observability model][4]
- [Power System deployment workflow][5]
- [Critical System Kafka Integration][6]

## 10. Contributors

Junhao Chen ([@ImperialHong][7]) added the Power System prototype to the
CyberLab.


[1]: https://github.com/UAdelaide/CyberLab/blob/main/containers/power_system/README.md
[2]: https://github.com/UAdelaide/CyberLab/blob/main/containers/power_system/docs/power-system-prototype-design.md
[3]: https://github.com/UAdelaide/CyberLab/blob/main/containers/power_system/docs/core-system-json-protocol.md
[4]: https://github.com/UAdelaide/CyberLab/blob/main/containers/power_system/docs/power-system-observability.md
[5]: https://github.com/UAdelaide/CyberLab/blob/main/containers/power_system/docs/power-system-ansible-workflow.md
[6]: https://github.com/UAdelaide/CyberLab/wiki/Critical-System-Kafka-Integration
[7]: https://github.com/ImperialHong
