# Orion-HQ

## 1. Project Summary

Orion-HQ is the cloud control plane for fleet-level Orion operations.

- It is split into a FastAPI/PostgreSQL human-facing API and an independent Python gRPC agent gateway.
- The API owns tenant-scoped users and devices, session and OIDC authentication, pairing credentials, telemetry, faults, alarms, logs, backups, update artifacts, audit records, and pending commands.
- The gateway owns the device-facing bidirectional `AgentLink.Connect` stream.
- It handles pairing handshakes, heartbeats, telemetry batches, fault events, log-upload requests, command acknowledgements, and stale-device sweeps.

The API is organized into routers, SQLAlchemy models, Pydantic schemas, services, and Alembic migrations.

- Device status is pushed to browsers through a WebSocket subscription.
- Storage helpers issue signed URLs for log and backup payloads.
- Tenant scoping is enforced at the router/service boundary and explicitly tested.
- The gateway has generated protobuf bindings, a connection manager, frame handlers, configuration, and a command-line test client.
- HQ deliberately uses its own cloud-agent protocol rather than the Orion-HAL/Orion-Client contract.

Docker Compose infrastructure supplies PostgreSQL and Keycloak; seed scripts, Make targets, and validation helpers support development.

- API tests cover RBAC, OIDC, pairing-token rotation, device ordering, telemetry and alarm round trips, phase-three resources, and cross-tenant isolation.
- Gateway tests cover pairing, frames, and stale sweeps.
- This README reports code and tests, not production cloud deployment, identity-provider operation, or a live fleet.

## 2. Feature Status

| # | Name / Description | Implemented? | Test Coverage? | Manual Test |
|---:|---|---|---|---|
| 1 | FastAPI REST API with async PostgreSQL | Implemented | Partial: API pytest suite | [ ] Manual verification required |
| 2 | Session, OIDC, RBAC, bearer credentials | Implemented | Covered by RBAC/OIDC/pairing tests | [ ] Manual verification required |
| 3 | Tenant-scoped devices and pairing | Implemented | Covered by device, pairing, isolation tests | [ ] Manual verification required |
| 4 | Telemetry, faults, and reported alarms | Implemented | Covered by telemetry, phase-three, alarm tests | [ ] Manual verification required |
| 5 | Logs, backups, signed URLs, audit records | Implemented | Partial: storage integration unverified | [ ] Manual verification required |
| 6 | Update artifacts, rollouts, pending commands | Partial | Unknown: models and services present | [ ] Manual verification required |
| 7 | Device status WebSocket broadcast | Implemented | Partial: browser integration unverified | [ ] Manual verification required |
| 8 | Bidirectional gRPC agent gateway | Implemented | Covered by pairing/frame tests | [ ] Manual verification required |
| 9 | Heartbeats and stale-device sweep | Implemented | Covered by `test_stale_sweep.py` | [ ] Manual verification required |
| 10 | Alembic, Docker Compose, and Keycloak infra | Implemented | Unknown: environment-dependent | [ ] Manual verification required |

## 3. Build and Test

```bash
cd api && pytest
cd ../agent-gateway && pytest
```

See `PLAN.md`, `ARCHITECTURE.md`, and the package metadata.
