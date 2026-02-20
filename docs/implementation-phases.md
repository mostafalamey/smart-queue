# Smart Queue — Implementation Phases Tracker

Date: 2026-02-20
Source: `docs/smart-queue-plan.md`

This document converts the project plan into implementation phases that can be tracked with branch and PR workflow.

## Progress Legend
- Status values: `Not Started` | `In Progress` | `Blocked` | `In Review` | `Done`
- Update fields per phase as work proceeds:
  - Owner
  - Start Date
  - Target Date
  - Status
  - PR Link

---

## Phase 0 — Repository & Delivery Workflow Baseline
**Goal:** Establish source control, branching strategy, and review gate before feature development.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: In Progress
- PR Link: N/A (baseline)

**Deliverables**
- Initialize git repository at project root.
- Commit current scaffolding as baseline.
- Define branch strategy:
  - `main`: protected, reviewed merges only.
  - `feature/<area>-<short-name>` for implementation work.
  - Optional `hotfix/<name>` for urgent production fixes.
- Define PR checklist and merge criteria.

**Done Criteria**
- Initial scaffold committed to `main`.
- Team follows “1 feature = 1 branch = 1 PR”.
- No direct commits to `main` except emergency hotfix policy.

**Suggested branch names for this phase**
- `chore/repo-baseline`

---

## Phase 1 — Domain Model, RBAC, and Core Data Contracts
**Goal:** Lock core entities, RBAC scopes, and migration-ready schema boundaries.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- Define entities: Hospital, Department, Service, Counter/Station, User, Role, Ticket, TicketEvent, PriorityCategory, MessageTemplate, Device, IntegrationConfig, AuditLog.
- Define RBAC policy matrix for Admin / IT / Manager / Staff.
- Define constraints:
  - one active ticket per phone per service
  - manager scoped to exactly one department
  - one service per counter
- Define multilingual field strategy (Arabic/English).

**Done Criteria**
- ERD and RBAC matrix reviewed.
- Database migration plan approved.
- API contract stubs for auth + queue resources aligned with RBAC.

**Suggested branch name**
- `feature/domain-rbac-foundation`

---

## Phase 2 — Queue Engine (Transactional Rules)
**Goal:** Implement strict priority + FIFO queue mechanics and event lifecycle consistency.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- Atomic operations: call next, recall, skip/no-show, transfer, complete.
- Enforce ordering: priority first, FIFO within priority.
- Transfer behavior:
  - new destination ticket number/prefix
  - destination queue insertion at end
  - preserve priority by default
- Ticket events persisted for all lifecycle changes.

**Done Criteria**
- Concurrency-safe call-next behavior validated.
- No manual queue reorder path exists.
- Ticket history fully reconstructs timeline.

**Suggested branch name**
- `feature/queue-engine-core`

---

## Phase 3 — Backend Services (API, Auth, Realtime, Jobs)
**Goal:** Deliver production-ready backend foundations for all clients.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- API service (NestJS + Prisma + PostgreSQL).
- Authentication:
  - email/password
  - Argon2id password hashing
  - short access token + rotating refresh token
- WebSocket broadcasting for queue and now-serving.
- Redis + BullMQ jobs for retries/scheduled tasks.
- Admin config endpoints (resets, templates, mappings, retention settings).

**Done Criteria**
- Auth and RBAC enforced server-side on all protected endpoints.
- Realtime queue updates available for teller/signage clients.
- Scheduled jobs run and recover from transient failures.

**Suggested branch name**
- `feature/backend-core-services`

---

## Phase 4 — Patient Channels (Kiosk + Patient PWA)
**Goal:** Enable ticket issuance and status flow for patients via web channels.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- Kiosk UI flows:
  - reception mode
  - department-locked mode
  - mandatory phone entry
- Ticket print payload generation (includes QR for WhatsApp opt-in).
- Patient PWA ticket + status flows.
- Offline handling messages (no issuance while backend unavailable).

**Done Criteria**
- Kiosk and patient flows create valid tickets under duplicate constraints.
- Printed payload includes required queue snapshot fields.
- Offline states are explicit and safe.

**Suggested branch name**
- `feature/patient-kiosk-pwa`

---

## Phase 5 — Staff Channels (Admin Web + Teller Desktop)
**Goal:** Deliver role-specific operations for Admin/IT/Manager and teller execution actions.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- Admin app sections by role:
  - Admin: full access
  - IT: User Experience + Mapping
  - Manager: Queue Control + Analytics (single department scope)
- Teller desktop app:
  - login
  - counter binding recognition via Device ID
  - actions: call next, recall, skip/no-show, transfer, complete
  - keyboard shortcut support

**Done Criteria**
- Role-based navigation and API access both enforced.
- Teller flow operates without manual counter/service switching.
- Queue-control priority changes blocked when ticket already called.

**Suggested branch names**
- `feature/admin-rbac-ui`
- `feature/teller-desktop-v1`

---

## Phase 6 — Display & Announcements
**Goal:** Deliver live display experiences for signage and per-counter screens, with optional LED/TTS adapters.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- Browser-based signage views per department zone.
- Per-teller now-serving screen.
- Adapter boundary for non-browser LED displays.
- Trigger model for call/recall announcements (Arabic then English), local/offline capable.

**Done Criteria**
- Live now-serving updates are stable and low-latency on LAN.
- Zoning rules respected for visual and audio outputs.
- Adapter path documented for vendor-specific LED protocols.

**Suggested branch name**
- `feature/signage-and-announcements`

---

## Phase 7 — UltraMessage WhatsApp Integration
**Goal:** Implement secure menu-driven WhatsApp issuance and notifications.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- Integration gateway boundary for inbound webhooks/outbound sends.
- Menu/button chatbot flows for issue/status/cancel.
- Enforce sender-phone ownership for all ticket commands.
- Lifecycle event notifications with per-event, per-language templates.
- Nearing-turn notifications using priority-aware people-ahead counts.
- Delivery logs and correlation to ticket lifecycle.

**Done Criteria**
- No free-text parsing dependency in core actions.
- `STATUS` and `CANCEL` behavior matches spec.
- Outage fallback response implemented.

**Suggested branch name**
- `feature/ultramessage-integration`

---

## Phase 8 — Admin Operational Tools (Deep Configuration)
**Goal:** Complete operational configuration surfaces and bindings in Admin app.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- Organization metadata + user lifecycle management.
- Department/service tree management with ticket prefix config.
- Mapping tools:
  - kiosk device config
  - counter/PC binding
  - signage zoning
- User Experience templates and multilingual content controls.

**Done Criteria**
- Admin/IT/Manager capabilities align exactly to role scopes.
- Device mappings persist and drive runtime behavior.

**Suggested branch name**
- `feature/admin-ops-tools`

---

## Phase 9 — Analytics, Retention, and Reporting Readiness
**Goal:** Ship operational metrics, predictive-ready data shape, and retention controls.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- KPI computation: wait, service time, throughput, no-show, transfers, peaks.
- Department-scoped analytics for managers.
- Retention policy settings and purge jobs.
- Optional aggregated reporting sync contract (deferred implementation acceptable).

**Done Criteria**
- Metrics align with ticket lifecycle events.
- Purge jobs respect configured retention windows.
- Department scope enforced in manager analytics.

**Suggested branch name**
- `feature/analytics-retention`

---

## Phase 10 — Packaging, Deployment, and Operations
**Goal:** Productionize on-prem deployment with repeatable install/upgrade/backup paths.

**Tracking**
- Owner: 
- Start Date: 
- Target Date: 
- Status: Not Started
- PR Link: 

**Deliverables**
- Docker Compose baseline for required services.
- Secrets and environment strategy.
- Backup/restore playbook for PostgreSQL.
- Upgrade/rollback runbook.
- Deployment validation checklist.

**Done Criteria**
- Fresh install succeeds on clean Linux host.
- Backup/restore drill verified.
- Upgrade and rollback drills verified.

**Suggested branch name**
- `feature/deployment-compose`

---

## Cross-Phase Quality Gates
Apply to every feature branch before PR approval.

- Unit/integration tests updated for affected modules.
- RBAC and scope checks included in API/UI behavior.
- Arabic/English content coverage considered where applicable.
- Audit events generated for queue-changing actions.
- No direct merge to `main` without review.

---

## Branch & PR Workflow (Working Agreement)

### Branch Naming
- Feature: `feature/<area>-<short-name>`
- Chore: `chore/<short-name>`
- Hotfix: `hotfix/<short-name>`

Examples:
- `feature/queue-engine-core`
- `feature/ultramessage-integration`
- `chore/repo-baseline`

### PR Rules
- One PR per feature branch.
- Keep PR scoped to a single phase objective when possible.
- Required in PR description:
  - What changed
  - Why it changed
  - How it was tested
  - Any migration/deployment impact

### Merge Rules
- Prefer squash merge to keep history clean.
- Delete feature branch after merge.
- Main branch should stay releasable.

---

## Immediate Next Steps
1. Create the repository (local + GitHub remote).
2. Push baseline scaffold commit.
3. Start first feature branch from Phase 1 (`feature/domain-rbac-foundation`).
4. Open PR to `main` after phase deliverables are complete.
