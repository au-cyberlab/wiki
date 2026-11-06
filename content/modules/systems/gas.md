# Overview

The Gas System is a prototype subsystem for the CyberLab core infrastructure project. It simulates
gas supply, runs compact OpenPLC control logic, exposes a Flask API and dashboard, and reports
`gas0` telemetry to the Power System through the current `power-system/v1` JSON-RPC contract.

The system still uses simulated gas input, but it is now integrated with the Power System for
telemetry and advisory dispatch acknowledgement.

## 1. System Logic

The basic flow is:

1. A user sends a request from the UI or API.
2. The Flask server receives the request.
3. The gas handler writes gas values or command codes to OpenPLC through Modbus.
4. OpenPLC runs the gas control logic.
5. The gas handler reads the PLC status back.
6. The Python service maps that status to CyberLab-facing kW values.
7. The API returns the latest gas status and power output.
8. When Power integration is enabled, Gas sends `gas0` telemetry to Power System and acknowledges
   active `GasRequestedPowerKw` dispatch requests.
9. The UI displays the current gas data.

OpenPLC decides the gas status. The Python service maps that status to externally reported kW
values using environment variables:

| Gas status | Meaning | Default reported kW |
| --- | --- | ---: |
| `online` | Gas supply normal | `100` |
| `degraded` | Gas supply reduced | `50` |
| `offline` | Gas supply too low | `0` |
| `shutdown` | Manual or external shutdown | `0` |
| `fault` | Gas system fault | `0` |

## 2. Ports

| Port | Purpose |
| ---: | --- |
| `9002` | Host-facing Gas API and dashboard |
| `5001` | Container-internal Flask API and dashboard |
| `1502` | Modbus / OpenPLC communication |
| `43628` | OpenPLC runtime support socket |

For local browser access through a jumpbox:

```bash
ssh -L 9002:<gas-server-ip>:9002 <jumpbox-user>@<jumpbox-ip>
```

Then open:

```text
http://localhost:9002/
```

## 3. API

Main endpoints:

- `GET /`
- `GET /api/heartbeat`
- `GET /api/gas_state`
- `GET /api/gas_status`
- `GET /api/outgoing_messages`
- `GET /api/integration_status`
- `POST /api/simulate`
- `POST /api/gas_set`
- `POST /api/recover`
- `POST /api/power_down`
- `POST /api/external_update`
- `POST /api/send_to_power`

Deployment smoke checks:

```bash
curl -s http://localhost:9002/api/heartbeat
curl -s -X POST http://localhost:9002/api/simulate
curl -s http://localhost:9002/api/gas_status
curl -s http://localhost:9002/api/integration_status
```

## 4. Power System Integration

When `POWER_INTEGRATION_ENABLED=true`, Gas sends JSON-RPC `currentData` telemetry to the Power
System endpoint on port `9001`.

Example telemetry:

```json
{
  "jsonrpc": "2.0",
  "id": "gas-telemetry-20260510T040000Z",
  "method": "currentData",
  "params": {
    "gas0/Status": "ONLINE",
    "gas0/AvailablePowerKw": 100.0,
    "gas0/Faulted": false,
    "gas0/LastUpdateTs": "2026-05-10T04:00:00Z"
  }
}
```

Gas reads the read-only `power0` coordination channels, especially:

- `DispatchRequestId`
- `GasRequestedPowerKw`
- `GasDispatchAckState`

If the active dispatch request targets Gas and the current state can satisfy the requested kW, Gas
sends `componentJsonApi.ackDispatch` as `ACCEPTED` and then `APPLIED`. If it cannot satisfy the
request, it sends `REJECTED`.

Important environment variables:

| Variable | Default | Description |
| --- | --- | --- |
| `POWER_INTEGRATION_ENABLED` | `false` locally, `true` in deployment | Enable Power System JSON-RPC integration |
| `POWER_JSONRPC_URL` | `http://power_system-server.power_system.lab:9001/jsonrpc` | Power System JSON-RPC endpoint |
| `POWER_REQUEST_TIMEOUT` | `3` | JSON-RPC timeout in seconds |
| `POWER_TELEMETRY_INTERVAL_SECONDS` | `10` | Telemetry and dispatch poll interval |
| `GAS_ONLINE_POWER_KW` | `100` | kW exposed when Gas is online |
| `GAS_DEGRADED_POWER_KW` | `50` | kW exposed when Gas is degraded |
| `GAS_OFFLINE_POWER_KW` | `0` | kW exposed when Gas is offline |

## 5. OpenPLC Overlay

The current repository does not store the full OpenPLC_v3 source tree. Gas-specific files are kept
under:

```text
containers/gas-system/gas_overlay/
```

During deployment, the Ansible playbook clones a clean OpenPLC_v3 repository and copies this overlay
into it before building the Gas Docker image.

Important files:

| File | Purpose |
| --- | --- |
| `gas_overlay/Dockerfile.gas` | Builds the minimal Gas OpenPLC container |
| `gas_overlay/webserver/gas_handler.py` | Bridges Flask, Modbus/OpenPLC, local messages, and Power telemetry |
| `gas_overlay/webserver/power_integration.py` | Sends `gas0` telemetry and handles dispatch ACKs |
| `gas_overlay/webserver/ui_server.py` | Flask API and UI server |
| `gas_overlay/webserver/st_files/gas_control.st` | PLC control logic |
| `gas_overlay/webserver/ui/` | Static dashboard assets |

## 6. Deployment

The Gas system is deployed with Ansible and Docker.

The deployment process is roughly:

1. Add the Power System host to a runtime Ansible group.
2. Clone the CyberLab repository.
3. Clone a clean OpenPLC_v3 repository.
4. Copy the Gas overlay files into the OpenPLC folder.
5. Build the Gas Docker image.
6. Start the Gas container with Power integration and Gas capacity environment variables.
7. Check that the API is responding.
