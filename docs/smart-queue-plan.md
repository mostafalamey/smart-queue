# Smart Queue — Project Plan (On-Prem Hospital Queue Platform)

Date: 2026-02-10

## Overview
Smart Queue is an on-prem queue management platform for a hospital group. Each hospital runs its own local instance (Linux + Docker Compose) with a local PostgreSQL database. Each installation serves **one hospital**. The queue is **per service** (Department → Service). Patients obtain tickets via **kiosk PWA**, **patient PWA (mobile web)**, and **WhatsApp (UltraMessage)**. Queue status is displayed on **digital signage** and **per-teller small screens**. Tellers use a dedicated **Teller desktop app** (Electron) to call next/skip/transfer and view queue metrics.

A central reporting layer is optional: each hospital can periodically sync aggregated analytics to a group-level reporting server when WAN connectivity is available.

## Key Requirements Captured
- **Deployment**: On-prem per hospital; Linux servers; Docker + Docker Compose.
- **Database**: PostgreSQL; database remains internal (no direct exposure).
- **WhatsApp**: UltraMessage integration; requires outbound internet from a dedicated integration boundary/gateway component.
- **WhatsApp capabilities**: Patients can create tickets via WhatsApp (UltraMessage) and receive status updates/notifications after ticket creation.
- **WhatsApp UX**: Strictly menu/button-driven (no free-text parsing) for reliability.
- **WhatsApp patient actions**: Patients can check ticket status and cancel tickets via WhatsApp using menu/buttons (buttons can send exact keywords like `STATUS` and `CANCEL`).
- **Patient cancel rule**: Patient cancellation is allowed only until the ticket is called; after a ticket is called, staff must resolve it (serve/transfer/no-show).
- **WhatsApp status/cancel UX**:
   - `STATUS` lists all active tickets for the phone number.
   - `CANCEL` shows a menu to cancel a specific ticket or cancel all (only for tickets not yet called).
- **WhatsApp security**: Commands operate only on tickets belonging to the sender phone number; no additional verification step.
- **WhatsApp notifications**: Smart Queue automatically sends WhatsApp updates (especially when the ticket is called and the counter is known).
- **WhatsApp events**: Send notifications for all ticket lifecycle events (created, called, serving started, completed, skipped/no-show, transferred, etc.), with templates configurable per event and per language.
- **Nearing-turn trigger**: Notify when there are **N people ahead** in the queue (N configurable per service/department).
- **Priority-aware counts**: “People ahead” counts only tickets with **equal or higher priority** than the patient’s ticket.
- **Opt-in scope**: Opt-in is associated to the patient phone number and applies to all of their tickets (until opt-out).
- **Ticket identity**: Phone number mandatory for ticket issuance; no OTP verification.
- **Duplicate rules**: One active ticket per phone number **per service** (can hold tickets across different services).
- **Ticket numbering**: Per service; resets daily; can also be manually reset by admin/manager.
- **Reset time**: Daily reset occurs at **midnight local time**.
- **Ticket format**: Ticket numbers include a per-service prefix (configurable), e.g., `LAB-023` or `A023`.
- **Prefix configuration**: Ticket prefix is set when creating the Service in the Departments Structure tree.
- **Priorities**: Priority categories supported; staff assigns/changes priority after ticket creation.
- **Priority levels (v1)**: Normal / VIP / Emergency.
- **Priority edits**: Admin/Manager can change priority only while the ticket is **not yet called**.
- **Ordering rules**: Strictly enforced by priority + FIFO only; no manual reordering of the queue.
- **Transfer**: Tickets can be transferred to any service in the hospital.
- **Transfer numbering**: On transfer, the ticket receives a **new number/prefix** in the destination service sequence; transfer history links original and new tickets for audit/analytics.
- **Transfer position**: Transferred tickets join the destination service queue in normal FIFO order (end of queue).
- **Transfer priority**: Preserve the ticket’s priority category by default.
- **Signage**: Existing signage devices can open LAN URLs; browser-based display views.
- **Above-teller display**: Per-counter “now serving” is shown on a small above-teller display; if the device is a simple LED panel (no browser), it will be driven via a hardware adapter (protocol TBD).
- **Above-teller content**: Counter number + now serving ticket.
- **Kiosk & mobile**: PWAs hosted on hospital servers.
- **Kiosk runtime**: For reliable ticket printing on Windows kiosks (silent printing, printer selection control), the kiosk UI may be packaged as a small **Electron kiosk app**.
- **Kiosk deployment**: Manual installation and updates (MSI/EXE) managed by hospital IT.
- **Admin app**: A single RBAC web app (PWA) for **Admin, IT, Manager**.
   - Admin: full access (including Organization + Departments Structure)
   - IT: User Experience + Mapping only
   - Manager: Queue Control + Analytics only (scoped to exactly one department)
   - Queue Control provides brief queue status plus ticket lookup/lock to change priority
- **Teller**: Windows PCs; teller uses a compact, minimal **Electron** desktop app (Staff role).
- **Teller deployment**: Manual installation and updates (MSI/EXE) managed by hospital IT.
- **Teller auth**: Tellers sign in with credentials inside the app and can log in from any PC; the **PC is bound to a counter/station** via the Admin tool to reduce clicks and errors.
- **Auth basics (defaults)**: Username is an email; passwords min length 12 with standard complexity; Admin performs password resets (temporary password + force-change).
- **Staff auth**: Managed entirely inside Smart Queue (no LDAP/Active Directory dependency for v1).
- **Device binding**: Counter/station bindings use an **app-generated Device ID** shown in the Teller app (entered by IT in the Admin app) to avoid issues with PC renames.
- **Counter/service rule**: One **service per counter**.
- **Teller workflow**: After login, teller operates the **service queue bound to their counter** (no service switching in v1).
- **Shortcuts & peripherals**: Teller actions support keyboard shortcuts so simple physical key peripherals can trigger Call Next / Recall / Skip / Complete / Transfer.
- **Languages**: Arabic + English.
- **Printing**: Kiosk must print physical tickets.
- **Kiosk printers**: Each kiosk has a connected thermal printer and prints via Windows printing (Windows spooler / installed printer).
- **Kiosk OS**: Windows (standardized).
- **Printed QR**: Printed tickets include a QR code that opens WhatsApp with a pre-configured opt-in message so the patient can receive automatic notifications.
- **Printed info**: Printed tickets include ticket number/prefix, counter/department context as applicable, and the queue position/people-ahead at print time (snapshot).
- **Wait time**: Kiosk and patient views can display an estimated wait time (and optionally print it), based on recent observed service rates and the current priority-aware queue.
- **Wait time scope**: Estimated wait time is calculated per service (v1).
- **Kiosk modes**:
   - **Reception mode**: patient selects department → service → enters phone → prints ticket.
   - **Department-locked mode**: kiosk is locked to one department; patient sees only services in that department with no back option.
- **Kiosk configuration**: Requires Admin/IT/Manager credentials to configure; kiosk remembers configuration across restarts.
- **Kiosk first run**: A configuration wizard is required on first run.
- **Audit/analytics**: Record full lifecycle including created, called attempts, served start, completed, skipped/no-show, transfer history.
- **Retention**: Configurable retention policy for phone numbers and ticket history.

## Architecture (High Level)
### On-Prem (Per Hospital)
- **Core API**: Node.js (TypeScript) API (PERN backend)
- **Framework**: NestJS
- **ORM/Migrations**: Prisma
- **Real-time**: WebSockets (Socket.IO) for live updates to teller view and displays
- **Database**: PostgreSQL
- **Reverse proxy**: Nginx/Traefik (LAN HTTPS termination)
- **Redis + BullMQ (recommended)**: async jobs/retries (UltraMessage send pipeline, webhook processing, scheduled tasks like retention purges)

### Frontend
- **Admin app**: React (PWA) using Refine Core (headless) for auth/RBAC/resources, with Tailwind + shadcn/ui
- **Teller app**: Electron desktop app hosting a compact React UI (Staff role)
- **Kiosk app**: Electron kiosk app (Windows) hosting the Kiosk UI for reliable printing and locked-down kiosk mode
- **Mode-specific web views**: Kiosk mode, patient mode, signage mode, teller small-screen mode

### Authentication & Login (Locked Approach)
- **User login**: Email + password against the Core API.
- **Password storage**: Argon2id hash (salted).
- **Session model**: short-lived access token + refresh token.
   - Access token: JWT, short TTL (e.g., 15 minutes).
   - Refresh token: long TTL (e.g., 30 days), rotated on each refresh.
- **RBAC enforcement**: always enforced server-side by the API.
- **Web (Admin PWA)**: refresh token stored as an HttpOnly secure cookie; access token refreshed via API as needed.
- **Desktop (Teller Electron)**: refresh token stored in OS secure storage (e.g., keytar on Windows); access token kept in memory and refreshed.
- **Non-human clients**:
   - **Signage** uses a device credential (enrollment token / API key) for read-only display endpoints.
   - **UltraMessage gateway** uses service-to-service auth (shared secret / HMAC) when calling internal APIs.

### Media Storage (User Profile Pictures)
Recommended for on-prem: store user avatars on local disk with a Docker volume.
- Upload via API (multipart): `POST /users/:id/avatar`
- Store file under a mounted volume (e.g., `data/uploads/avatars/{userId}/{uuid}.webp|.jpg`)
- Store metadata in Postgres (path/key, content type, updated time)
- Serve via an authenticated API route (RBAC-protected)

### Kiosk Printing Notes
- A kiosk PWA can trigger printing, but true “silent printing” (no dialog) typically requires kiosk browser/device policy configuration.
- Packaging the kiosk UI in Electron is the simplest reliable path for Windows kiosks: it can print silently to a configured printer and run in locked-down kiosk fullscreen.
- Keep printing architecture flexible: start with Electron kiosk app on Windows; keep the Kiosk UI as a web view for environments that don't need local printing.

### Kiosk Configuration Notes
- The kiosk app generates and displays a Device ID.
- On first run, a wizard configures: kiosk mode (Reception vs Department-locked), assigned department (if applicable), default printer, language defaults, and the server URL.
- Configuration is persisted locally (survives restarts) and can be changed only after entering Admin/IT/Manager credentials in a protected config screen.
- Optional remote configuration: Admin/IT can manage kiosk settings remotely in the Admin app by Device ID.
   - Use secure device enrollment (e.g., pairing code shown on kiosk, registered by IT) to bind the kiosk to the hospital server.
   - Kiosk periodically polls the server (or uses a WebSocket) to fetch updated configuration.
   - Remote changes are applied on kiosk restart or an explicit “Apply update” action (to avoid disrupting patient flows).
   - Local protected config remains available for break-glass scenarios.

### Kiosk Offline Behavior
- If the kiosk cannot reach the backend/API (and thus the database), it blocks ticket issuance and shows an offline message directing patients to reception.

### Patient PWA Offline Behavior
- If the patient PWA cannot reach the backend/API, it blocks ticket issuance and shows a clear offline message.
- For status viewing, the patient PWA can cache and show the last-known ticket status when temporarily offline (with a “last updated” timestamp).

### WhatsApp Integration Boundary
- Separate component/container responsible for:
   - Outbound HTTPS calls to UltraMessage
   - Receiving provider webhooks (on-prem)
   - Applying message templates (Arabic/English)
   - Logging delivery events
- Communicates with the internal API over restricted ports and a narrow, authenticated interface.
- Security note (default): the UltraMessage gateway is the only component exposed to the internet (DMZ/reverse proxy); the core API and database remain internal.

### Hardware Integrations (Per Hospital)
- **LED display adapter (optional)**: A small local service that subscribes to “now serving” events and pushes updates to the above-teller LED displays.
   - Connection/protocol depends on vendor: Serial (RS-232/RS-485), USB, or Ethernet/TCP.
   - The adapter should be implemented behind an interface so the vendor-specific driver can be swapped without changing the core system.

### Audible Announcements (Per Hospital)
- **Local TTS (offline)**: Provide an on-prem text-to-speech capability that does not require internet.
   - Implement as a small TTS service/container (Linux) using an offline engine (e.g., Piper) to generate audio files/streams.
   - Trigger announcements on key events (Call Next / Recall) with a configurable template (Arabic/English).
   - Announcement language: Arabic + English, Arabic first.
   - Announcement playback: digital signage player devices/PCs/boxes play the generated audio over connected speakers.
   - Zoning: each signage player is assigned to a department zone and only announces tickets for services within that department.

### Central Reporting (Optional)
- A separate server that receives **aggregated metrics** from each hospital.
- Sync is scheduled and retryable, tolerant to unreliable WAN.

## Plan (Execution Steps)
1. **Define the domain model and RBAC**
   - Entities: Hospital, Department, Service, TellerStation/Counter, User, Role, Ticket, TicketEvent, PriorityCategory, MessageTemplate, Device, IntegrationConfig, AuditLog.
    - Roles (initial): Admin, IT, Manager, Staff.
       - Admin: full access
       - IT: mapping/devices/integrations and operational settings (no access to organization metadata or department/service structure)
       - Manager: queue control + analytics for their assigned department only (exactly one department)
       - Staff: teller actions and queue visibility (scope-limited); assigned to a department

2. **Implement the queue engine with strict transactional rules**
   - One active ticket per phone per service.
   - Priority categories are served before normal; FIFO within each priority.
   - Atomic operations for call/recall/skip/transfer/complete.
   - Define **Skip** as a final **no-show** outcome (ticket removed from active queue), recorded in the event history.
    - Define **Recall** as re-announcing the currently called/serving ticket (does not change ordering), recorded in the event history.
       - Recall triggers signage/LED updates and re-sends the WhatsApp “called” notification.
   - Full event history stored in TicketEvent.

3. **Build the backend services**
   - Node.js (TypeScript) API
   - WebSocket gateway for queue state and “now serving” broadcasts
   - Admin endpoints for configuration, resets, RBAC, templates

4. **Build patient UIs as PWAs**
   - **Kiosk PWA**: kiosk mode UI, mandatory phone entry, prints physical ticket.
   - **Patient PWA**: QR/link entry, take ticket, view queue status, WhatsApp opt-in/handoff.

5. **Build the Admin RBAC app (web) and the Teller desktop app**
   - **Admin RBAC app**: React PWA with role-based screens (Admin / IT / Manager)
   - **Teller app**: Electron desktop app with a compact React UI (Staff role)
   - Teller signs in with credentials; app receives a short-lived access token; API enforces RBAC and scope
   - Counter/station is identified from the **PC binding** configured in the Admin/IT view (no manual counter selection)
   - Teller operates the service queue bound to their counter (one service per counter in v1)
   - Actions: Call Next, Recall, Skip (No-show), Transfer, Complete
   - Keyboard shortcuts for Call Next / Recall / Skip / Complete / Transfer to support physical key peripherals (USB HID keystrokes)
   - If the above-teller LED display is connected to the teller PC (Serial/USB/Ethernet): the Teller app can drive it directly via a device adapter module

6. **Build display views (browser-based)**
   - **Digital signage view(s)**: department-zone dashboards showing multiple counters/services at once (now serving + next N).
   - **Per-teller small screen view**: “Now Serving” for a specific counter.
   - If above-teller devices are non-browser LED panels: integrate via the LED display adapter service instead of a web view
   - Audible announcements: signage/player clients subscribe to call/recall events and play local TTS audio (no internet)

7. **Implement UltraMessage integration**
   - Inbound webhooks terminate at the on-prem UltraMessage gateway (DMZ/reverse proxy); gateway also performs outbound calls to UltraMessage
   - WhatsApp ticket issuance flow (chatbot/menu-driven) to select department/service and generate a ticket
   - Template-driven messaging (Arabic/English)
   - Status queries and proactive notifications (e.g., ticket created, nearing turn, called, transferred)
   - For tickets created outside WhatsApp (kiosk/PWA): capture an explicit WhatsApp opt-in so outbound notifications are allowed
     - Printed ticket QR deep-links to WhatsApp (e.g., `wa.me/...` with prefilled text)
       - Prefilled text should include a ticket reference or short-lived signed token so Smart Queue can safely associate the opt-in with the right phone number
       - Support opt-out (menu/button-driven) to stop future notifications
   - Store delivery logs/events and correlate to tickets
   - Outage behavior: if the backend/API is unavailable, respond with a clear “system temporarily unavailable, please try again later” message

8. **Implement the Admin/IT/Manager tools inside the Admin RBAC app**
   - Admin: configure users/roles, global settings
   - IT: configure **Mapping** (devices/counters/PC bindings, signage zoning), integrations (UltraMessage), operational settings
   - Manager: queue control + analytics access (department-scoped)
   - Admin: full access to all sections including Organization metadata and Departments Structure
   - QR code generation (PWA entry links + WhatsApp entry)

9. **Implement analytics + retention controls**
   - Wait time metrics, throughput, no-shows, transfer rates, peak hours
   - Configurable retention settings and purge jobs

10. **Package and deploy with Docker Compose**
   - Containers: api, postgres, reverse proxy, ultramessage-gateway, redis (BullMQ)
   - Optional: separate worker container if we split job processing from the API process
   - Secrets management
   - Backup/restore playbook for Postgres

## Verification Strategy
- **Correctness tests** for queue behavior:
  - Concurrency/race-condition safety for “call next”
  - Priority ordering correctness
  - Transfer history integrity
  - One-active-ticket-per-phone-per-service enforcement

- **End-to-end flows**:
  - Kiosk issuance + physical printing
  - Teller call/skip/transfer + live signage updates
  - WhatsApp notification triggers and templates

- **Load/performance**:
  - Peak ticket issuance
  - Concurrent teller operations
  - 30 signage screens polling/connected

- **Deployment validation**:
  - Fresh install on a clean Linux host
  - Upgrade/rollback drill
  - Backup/restore drill

## Open Items / Assumptions
- Physical button interface is not decided; default assumption is USB HID keystrokes via teller app shortcuts.
- Existing digital signage can open LAN URLs (assumed supported).
- Above-teller LED display vendor/model and connection/protocol are TBD; implement as an adapter/driver integration.
- Central reporting sync is optional and can be deferred until the on-prem core is stable.

## Notes on PERN Feasibility
- Converting the stack to PERN is feasible for all web surfaces (admin/IT/manager/staff, kiosk, patient, signage).
