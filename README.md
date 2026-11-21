# Karkain CRM

A local-first, CLI/TUI customer relationship manager with scriptable bulk operations and explicit sync contracts to Karkain ERP.

## Status

| Item | Status |
|---|---|
| Research | COMPLETE |
| Product design | COMPLETE |
| Repository creation | COMPLETE |
| Implementation | NOT STARTED |
| Compile/execute verification of the application | NOT STARTED (toolchain itself VERIFIED on Karkain v1.2.0) |

## What it does

Karkain CRM manages the front-office record: leads, contacts, accounts, opportunities, activities and pipeline stages. It stores everything locally in the embedded `db2` database and exposes a scriptable command surface so bulk lead import, activity logging and pipeline queries can be automated from a shell or scheduled job. A flat, versioned sync contract publishes customer and opportunity summaries to Karkain ERP; the CRM owns lead-to-opportunity data and does not absorb ERP's organization hierarchy, Accounting's ledger or Inventory's stock records. Interaction is through a command-line interface and a text UI; there is no graphical desktop client and no browser dependency.

## Who it serves

Small field and inside-sales teams that find GUI-centric CRMs heavy and metered. Typical users are sales operators who need offline capture, reproducible batch operations and data sovereignty, and systems maintainers who want predictable flat licensing without per-seat or per-contact charges. It suits deterministic, offline or air-gapped environments where a local store is preferable to a SaaS subscription.

## Planned features

- **Contact management** — leads, contacts, accounts and interaction history (planned); deduplication rules (planned).
- **Pipeline** — opportunities, stages and probability tracking (planned); activity and task logging (planned).
- **Scripting** — bulk import/export of leads and activities (planned); batch commands for repeated operations (planned); query commands with text/JSON output (planned).
- **ERP sync** — versioned, explicit summary contracts to Karkain ERP (planned); replayable change events (planned).
- **Reporting** — pipeline snapshots and conversion counts (planned); exportable summaries (planned).
- **Administration** — roles for command access (planned); local audit log (planned); configuration profiles (planned).

## Explicit non-goals

- Re-implementing ERP organization hierarchies, workflows or master-data governance; Accounting journals or tax; Inventory SKUs or stock tracking.
- Native GUI desktop application in v1.2.0.
- External email delivery or cloud AI assistants; TLS and WebSocket are unavailable in v1.2.0.
- External SQL engines with JOIN reporting, or browser/mobile clients.
- Per-seat or per-contact metering; licensing is flat and Apache-2.0.

## Karkain foundation

Exclusive foundation: Karkain Language v1.2.0. All application source will be Karkain. No other implementation language. No compiler modifications.

## Platform support status

Windows x86-64 toolchain VERIFIED (compile to native executable and execute). Linux cross-target output PRESENT but execution NOT VERIFIED. macOS NOT VERIFIED. These describe the toolchain, not this application, whose compilation is NOT STARTED.

## Build and run status

No application binary exists yet. The intended generic commands are `karkain build src/main.kark -o karkain-crm` and `karkain run src/main.kark`. These become active only when implementation starts; until then they are informational and have not been executed for this application.

## Known blockers

- Native GUI toolkit BLOCKED — CLI/TUI-first design; GUI is a planned, blocked layer.
- Full JSON DOM/schema PARTIAL — sync payloads limited to simple field extraction.
- TLS BLOCKED and WebSocket BLOCKED — no external email/AI integration; local file/CLI exchange only.
- `system`/`env`/`process`/`time` PARTIAL shell wrappers — must be probed before any date/process claim.
- Embedded `db2` SQL subset with no JOINs; reporting uses precomputed summaries.
- OS atomic-write/file locking PARTIAL — durability of change-event checkpoints unverified.

## Repository

GitHub: https://github.com/ajit-ai/Karkain-CRM — issues: https://github.com/ajit-ai/Karkain-CRM/issues — License: Apache-2.0.
