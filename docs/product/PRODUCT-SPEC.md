# Karkain CRM — Product Specification

**Document status:** Design COMPLETE. Research = COMPLETE. Repository creation = COMPLETE. Application implementation = NOT STARTED. Application compile/execute verification = NOT STARTED (toolchain path itself VERIFIED on Karkain v1.2.0).


## A. Product definition

**Product name:** Karkain CRM.

**Purpose/problem:** GUI-centric CRMs are browser-dependent, metered per seat or contact, and poor at offline capture and scriptable batch work. Small field teams need a dependable local record of leads, contacts and opportunities that can be automated and synchronized deliberately rather than streamed to a SaaS platform.

**Vision:** The local-first, scriptable front-office record of the Karkain portfolio — a small deterministic service storing customer relationships locally and exchanging explicit, versioned summaries with Karkain ERP.

**Users/segments:** Small sales teams, field operators with intermittent connectivity, integrators and maintainers wanting reproducible bulk operations, data sovereignty and flat licensing. Buyers for whom enterprise CRM depth and per-user fees are disproportionate.

**Usage scenarios:** Log a lead and activity offline; import a batch of contacts; publish opportunity summaries to ERP; convert a lead and advance pipeline stages.

**Positioning/value proposition:** A CLI/TUI CRM service on embedded `db2` with local-first storage and explicit JSON-sync contracts to ERP. Value: offline capture, batch scripting, predictable flat cost, no marketplace lock-in.

**Non-goals/boundaries:** CRM does not duplicate ERP organization hierarchy/workflows, Accounting ledgers/tax, or Inventory stock. No GUI in v1.2.0; no external email/AI (TLS/WebSocket BLOCKED); no external SQL, browser/mobile clients or alternate language.

## B. Functional architecture

| Module | Responsibility |
|---|---|
| Leads & contacts | Leads, contact details, accounts, dedup rules |
| Opportunities | Stages, amount, probability, expected close |
| Activities | Tasks, calls, notes, interaction timeline |
| Pipeline | Stage definitions, conversion rules, snapshots |
| Query & scripting | CLI queries, filters, text/flat-JSON output |
| Import/export | Bulk lead/activity import/export, flat formats |
| Change log & sync | Append-only change events, versioned ERP contracts |
| Reporting | Precomputed pipeline/conversion summaries (no JOINs) |
| Administration | Roles, audit log, configuration profiles |

**Core workflows:** Create lead → qualify → convert → advance stage → close; log activity; import batch; export snapshot; publish change events.

**Admin workflows:** Define stage schema and roles; configure sync version; review audit log; archive/restore store; resolve sync rejections.

**Error/recovery:** Reject malformed imports per row; dead-letter invalid change events; replay idempotently; convert a lead only once.

**Roles/permissions:** Administrator (config), Sales operator (full CRUD), Reader (queries), Sync service (publish only), enforced at CLI/TUI layer; not a security certification.

**Configuration/customization:** Local profile defines stages, dedup rules and sync version; customization via config and Karkain scripts, no compiler changes.

**Import/export:** Flat versioned line records and simple `json` field extraction; deterministic, re-importable; no full JSON DOM assumed.

**Integration:** Versioned summary contracts to ERP via local CLI/file drop; no TLS/WebSocket/external API. CRM owns customer/opportunity records; ERP consumes summaries only.

## C. Competitor analysis

Competitors: **Salesforce (Sales Cloud)**, **Zoho CRM**, **HubSpot CRM**. Evidence and sources are in `docs/competitor-analysis/COMPETITORS.md`; labels match that document.

**Positioning:** [VF] Salesforce calls itself "#1 AI CRM" with Starter/Pro/Core suites. [VF] Zoho offers free-forever for 3 users then Standard/Professional/Enterprise/Ultimate plus "CRM for Everyone." [VF] HubSpot offers free CRM tools then Starter/Professional/Enterprise seats. [RI] Zoho and HubSpot compete hardest on SMB price; Salesforce on enterprise depth.

| Capability | Karkain CRM [P] | Salesforce | Zoho | HubSpot |
|---|---|---|---|---|
| CLI/TUI-first | Yes | No | No | No |
| Offline local store (db2) | Yes | No | No | No |
| Free tier | [P] flat license | Trial | 3 users forever | 2 users |
| AI assistant | [P] optional/pluggable | Agentforce/Einstein | Zia | HubSpot AI |
| Marketplace | [P] no lock-in | AppExchange (500+) | Zoho marketplace | App marketplace |
| Field-level encryption | [UNKNOWN] scope | Available | Higher tiers | Higher tiers |

**Platform/deployment:** [VF] All three are cloud/SaaS with web UIs and mobile apps; Salesforce/Zoho integrate with Gmail/Outlook. [RI] No offline-first CLI option among them — the gap Karkain targets with a local CLI/TUI service on `db2`. Karkain field-level encryption is [UNKNOWN] and not claimed.

**Licensing (verified only):** [VF] Salesforce: Starter $25, Pro $100, Core $195 USD/user/month billed annually. [VF] Zoho: free for 3 users; CRM Plus from $20/user/month; per-tier Standard/Professional/Enterprise/Ultimate figures **UNKNOWN — not verified**. [VF] HubSpot: free (2 users), Starter $20/seat/mo, Professional $57/mo/seat, Enterprise $98/mo/seat billed annually (promotional "was" prices shown). Karkain licensing: [P] flat Apache-2.0. No prices invented.

**Strengths/limitations:** [VF/R] Salesforce: unmatched ecosystem/AI, highest cost/complexity. Zoho: low entry cost and depth, but tier-gated AI/admin/security and 3-user cap. HubSpot: easy UX and free entry, but seat/contact limits and automation paywall.

**Market gaps:** [RI] Small field teams needing offline capture, scriptable/batch operations, data sovereignty and predictable flat pricing without metering.

**Differentiation:** [P] CLI/TUI CRM with local `db2` storage and explicit JSON-sync contracts to ERP, a scriptable bulk-operation surface and no marketplace lock-in.

**Deliberately not copied:** App marketplaces, embedded AI assistants, low-code builders, browser/mobile clients, TLS-dependent email sync, per-seat/contact metering.

## D. Technical architecture

**Application/boundaries:** A dispatcher fronts leads/contacts, opportunities, activities, pipeline, query, import/export, change-log/sync and reporting; each owns its `db2` tables with narrow interfaces. Single executable as one-shot CLI, TUI loop or minimal local service; sync to ERP by local CLI/file drop. No non-Karkain in-process extensions.

**UI architecture:** The UI is **CLI/TUI/service-first because Karkain v1.2.0 has no native GUI toolkit (BLOCKED)**. A GUI is a planned layer contingent on ecosystem capability, not part of v1.2.0. Every TUI action has a CLI equivalent.

**Domain model:** Lead, Contact, Account, Opportunity, Activity, PipelineStage, ChangeEvent, SyncContractVersion, ExceptionItem, Role, AuditEntry — front-office data only, referencing ERP ids without copying ERP master data.

**Storage strategy:** Embedded `db2` SQL subset for records plus append-only change-event log; no external SQL engine; summaries materialized sequentially, no JOINs.

**API/events:** Stable CLI subcommands plus minimal `http` building-block request/response; no routing/concurrent-server framework. Change events carry id, entity type, version and flat payload; handlers idempotent by id; replay safe; invalid events dead-lettered.

**Logging/config/extensions:** Structured text logs with run id and import/export/publish counters plus local audit trail (planned). Local profile selects stages, dedup and sync version via `config`/`io` once probed, no machine-specific paths. Extensions are Karkain-native; optional AI [P] would be a local pluggable adapter, not a cloud dependency. No FFI planned.

## E. Data design

**Entities/ownership:** Lead, Contact, Account, Opportunity, Activity, PipelineStage, ChangeEvent, SyncContractVersion, ExceptionItem, AuditEntry. Activities belong to contacts; a lead converts at most once; events reference the entity version producing them. CRM owns front-office records; ERP owns coordination data.

**Validation/lifecycle:** Required fields, unique ids, stage transitions and version compatibility enforced in code; dedup rules deterministic. Lead: new→qualified→converted/disqualified; Opportunity: open→stages→won/lost; Event: created→published→applied→archived.

**Persistence/transactions:** `db2` tables plus append-only change log. `db_begin`/`db_commit`/`db_rollback` exist in v1.2.0, but **OS-level atomic write is unverified**, so durability is limited to the embedded database; checkpoints reconciled on startup.

**Backup/migration/IO/audit:** Stop service, copy store and log as pair, restore by replacing both; no online snapshot. Explicit version integers with adjacent-version Karkain migration routines; no pre-v1.2.0 schema. Deterministic flat format and simple `json` extraction. Mutating commands append an AuditEntry with actor, command and timestamp (once `time` is verified).

## F. Karkain implementation architecture

**Capabilities (status per register):** Compiler v1.2.0 VERIFIED; lint VERIFIED; C-free native Windows backend VERIFIED with Go engine (default `kcc` BLOCKED — environment); `db` PRESENT/PARTIAL; `json` PARTIAL; `io`/`net`/`http`/`crypto`/`encoding`/`async`/`collections`/`generics` PRESENT; `filesystem` PARTIAL; `system`/`env`/`process`/`time` PARTIAL shell wrappers needing probes. No GUI module.

**Runtime/packaging:** Karkain v1.2.0 only, no compiler modification; Go front-end with C-free native target. Required ecosystem component: Karkain ERP with a defined sync contract, which must exist before sync work. One native executable in a directory package with default config and empty data directory; no machine-specific paths.

**Blockers:** (1) GUI toolkit BLOCKER — CLI/TUI only, GUI a blocked planned layer; (2) full JSON DOM PARTIAL — flat payloads; (3) TLS and WebSocket BLOCKED — no external email/AI, local exchange only; (4) `system`/`env`/`time` PARTIAL — probe before date/process claims; (5) OS atomic write PARTIAL — checkpoint durability unverified. Each recorded, never resolved by changing Karkain or using another language.

**Native/FFI:** None for MVP; any future FFI must be a justified Karkain-compatible native interface, never a replacement language.

## G. Non-functional requirements

Targets give workload and measurement method; no arbitrary numbers asserted. Practical machine ~4 GB RAM.

| Area | Target / method |
|---|---|
| Performance | Import 1,000 leads, time with `bench`, expect linear scaling |
| Memory/CPU | Profile import with `prof`; steady state within ~4 GB budget |
| Scalability | 10× batch, record time ratio; document `db2` ceiling |
| Reliability | Inject malformed rows and abrupt stop; dead-letter intact, replay idempotent |
| Security/privacy | Role command checks, no secret logging, no telemetry; not a certification |
| Portability | Cross-build Linux; execution UNVERIFIED on this host |
| Maintainability/accessibility | Interfaces documented and unit-tested; text-only keyboard TUI, redirectable output |
| Observability/recovery | Run id, counters, logs; kill mid-import, no partial/duplicate records |

## H. Compatibility and portability

Windows x86-64 toolchain VERIFIED; Linux cross-target output PRESENT but execution NOT VERIFIED; macOS NOT VERIFIED. All source targets v1.2.0 with Go front-end/C-free backend; default `kcc` unusable here (environment blocker). Native executable plus directory package. Install/uninstall by directory operations after export. Adjacent-version config/data migration; export before upgrade. Backward compatibility not required before v1.2.0. Upgrade/rollback via stop, backup pair, replace, migrate, verify; restore on failure. Unsupported: GUI, TLS, WebSocket, external SQL, concurrent HTTP server, non-Windows execution verification, field-level encryption claims.

## I. Testing strategy

All tests **PLANNED**; none have been run for this application.

- **Unit:** Conversion, dedup, stage transitions, contract validation — deterministic criteria.
- **Integration:** `db2` persistence + change log + sync round-trip — survives restart, no duplicates.
- **E2E:** Lead-to-opportunity with stub ERP consumer — expected summary published.
- **Platform-specific:** Windows native compile-and-run exit 0; other platforms cross-build only.
- **Security:** Role restrictions, no secret leakage — unauthorized commands rejected.
- **Performance:** Batch import with `bench` — measured, documented scaling.
- **Regression/compatibility:** Replay fixtures and migrate adjacent versions — identical outputs, lossless.
- **Actual-compiled-executable:** Karkain v1.2.0 build, run native binary — expected stdout.
- **Failure/recovery:** Kill mid-import — no partial/duplicate records.

## J. Implementation roadmap

**MVP:** Leads/contacts/accounts, change log, contract registry, CLI queries, one sync summary. **Core:** Opportunities, activities, pipeline, import/export, roles/audit, backup/restore. **Advanced:** Replay tooling, optional local AI adapter [P], richer reporting, minimal local service. **Production-readiness:** Durability verification, observability, published limits, cross-target builds, full suite.

**Dependencies/gates:** Contracts before sync; records before change log; change log before workflows; durability probes before production readiness. Each phase exits only on passing tests and a verified actual compiled executable. No dates.

**Risks/mitigations:** PARTIAL JSON — flat contracts; TLS/WebSocket BLOCKED — local exchange; atomic write unverified — conservative checkpoints; ERP contract undefined — define first; GUI BLOCKED — CLI/TUI only. No completion claims.
