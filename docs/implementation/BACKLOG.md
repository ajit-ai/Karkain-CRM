# Karkain CRM — First Implementation Backlog

## First vertical slice

**Slice goal:** Prove a complete lead-to-opportunity workflow: create a lead, log an activity, convert it once to an opportunity, and publish a flat change event to the ERP sync contract — real persisted work, not empty screens.

**Karkain modules/files expected:** `src/main.kark` (dispatcher), `src/leads.kark`, `src/opportunities.kark`, `src/activities.kark`, `src/change_events.kark`, `src/contracts.kark`, `src/store.kark` (using `db`, `io`, `json` PARTIAL, `collections`, `encoding`).

**Exact input:** `karkain-crm lead create --id lead-7 --name "Ada Corp" --contact "ada@example.test"` followed by `karkain-crm opportunity convert --lead lead-7 --amount 1200.00`, then `karkain-crm sync publish --contract opportunity.v1`.

**Exact expected observable output:** `Created lead-7`; `Converted lead-7 to opp-7 amount=1200.00`; `Published evt-1 contract=opportunity.v1 entity=opp-7`. A second convert prints `Already converted` and creates no new opportunity. `karkain-crm query opportunities` lists exactly one row.

**Verification command pattern (Karkain v1.2.0):** With the Go front-end engine selected, build/run with the verified C-free native target —
```
karkain build src/main.kark --target native-x86_64-windows -o karkain-crm
.\karkain-crm lead create --id lead-7 --name "Ada Corp" --contact "ada@example.test"
.\karkain-crm opportunity convert --lead lead-7 --amount 1200.00
.\karkain-crm sync publish --contract opportunity.v1
.\karkain-crm query opportunities
```
Build must exit 0 and the executable must print the exact lines above. No test has been run yet; this is the planned verification pattern.

## Backlog items

| ID | Feature | Files/modules | Dependencies | Acceptance criteria | Tests | Complexity | Blocking Karkain capability | Definition of done |
|---|---|---|---|---|---|---|---|---|
| CRM-001 | Command dispatcher & config profile | `main.kark`, `config_profile.kark` | None | All planned subcommands dispatch; no hard-coded paths | Unit, CLI smoke | S | `system`/`env` PARTIAL | Builds, help lists commands |
| CRM-002 | Leads, contacts & accounts CRUD | `leads.kark`, `store.kark` | CRM-001 | Unique ids, persisted in `db2`, queryable | Unit, integration | M | `db` PRESENT/PARTIAL | CRUD survives restart |
| CRM-003 | Activities & interaction timeline | `activities.kark` | CRM-002 | Activity linked to contact, ordered timeline | Unit | S | `time` PARTIAL wrapper | Timeline reproducible |
| CRM-004 | Opportunities & single-conversion rule | `opportunities.kark` | CRM-002 | Convert once; amount/stage stored | Unit, regression | M | None | Duplicate convert rejected |
| CRM-005 | Pipeline stages & snapshots | `pipeline.kark` | CRM-004 | Stage transitions validated; summary precomputed | Integration | M | `db2` no JOINs | Snapshot matches fixtures |
| CRM-006 | Change log & idempotent sync publish | `change_events.kark`, `contracts.kark` | CRM-002, CRM-004 | Flat event published; replay no duplicates | Integration, failure | M | `json` PARTIAL, atomic write PARTIAL | Replay safe |
| CRM-007 | Bulk import/export of leads & activities | `importexport.kark` | CRM-002, CRM-003 | Deterministic round-trip of flat records | Integration | M | `json` PARTIAL | Export re-imports identically |
| CRM-008 | Query & scripting commands | `query.kark` | CRM-002, CRM-005 | Filters and text output stable; script-friendly | Unit, regression | M | None | Output contract documented |
| CRM-009 | Roles & local audit log | `auth.kark`, `audit.kark` | CRM-001, CRM-002 | Unauthorized command rejected; mutations logged | Security, unit | M | `time` PARTIAL wrapper | Roles enforced |
| CRM-010 | Minimal local service loopback | `service.kark` | CRM-006 | Sequential request/response; no routing assumed | Integration, platform | L | HTTP framework BLOCKED, TLS BLOCKED | Local-only loopback works |

## Dependency order and gates

1. CRM-001 → 2. CRM-002 + CRM-003 (parallel) → 3. CRM-004 + CRM-007 (opportunities/IO) → 4. CRM-005 + CRM-006 (pipeline/sync) → 5. CRM-008 + CRM-009 (query/admin) → 6. CRM-010 (service). Gate between phases: the actual Karkain v1.2.0 compiled executable must be run and its expected output verified before advancing; all blockers (GUI, JSON, TLS/WebSocket, atomic write) remain recorded, never bypassed by changing Karkain.

## Status

All items: **NOT STARTED**. Implementation must not begin until all designs and repository initialization are complete. No tests have been run and no binary exists for this application.
