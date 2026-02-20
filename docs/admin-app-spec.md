# Smart Queue — Admin App Spec (RBAC Web App)

Date: 2026-02-19

This document describes the Admin App UI/UX and responsibilities.

Tech stack (locked): React PWA + Refine Core (headless) + Tailwind CSS + shadcn/ui.

## Sidebar Tabs
The sidebar includes these tabs:
1. Queue Control
2. Analytics
3. Organization
4. Departments Structure
5. User Experience
6. Mapping

### Roles
Roles in scope for the Admin App:
- **Admin**: can see everything
- **IT**: can see **User Experience** and **Mapping** only
- **Manager**: can see **Queue Control** and **Analytics** only (scoped to exactly one assigned department)

---

## 1) Queue Control
Purpose: let Admin/Manager see brief queue status and perform controlled ticket interventions (priority changes).

### Access
- Admin: all departments
- Manager: only their assigned department

### UI
- Department selector (Admin only)
- Service selector (scoped by selected department)
- Brief queue status widgets
  - e.g., waiting count, in-progress count, currently serving (per counter), average wait/service (today)
- Ticket lookup
  - By ticket number
  - By phone number
- Ticket details panel (read-only fields + audit trail preview)
- “Lock ticket” action (to prevent concurrent edits)
- Priority change UI (after locking)

### Behaviors
- Queue status can be live/near-live (polling or WebSocket).
- Priority changes are audited (who/when/previous→new priority).
- Ticket locks are time-bounded and released automatically on timeout.

### Restrictions
- No manual queue reordering beyond FIFO + priority.
- No teller execution actions here (Call Next / Recall / Skip / Transfer / Complete remain in Teller app).
- Cannot change ticket priority after the ticket has been **called**.

---

## 2) Analytics
Purpose: performance and operational analytics, with predictive insights.

### Access
- Admin: all departments
- Manager: only their assigned department

### Filters
- Time range
- Department

### KPI Cards / Metrics
- **Average Wait Time** — time from ticket creation to call
- **Average Service Time** — time from call to completion
- **Completion Rate**
- **Currently Waiting**
- Tickets issued today
- Tickets served
- In progress
- No show rate

### Historical Trends
- “No historical data available yet” placeholder until data exists

### Predictive Insights (Powered by historical analytics)
- **Wait Time Estimation** (with confidence indicator)
  - Analysis factors:
    - similar time periods analyzed
    - current queue length
    - historical average for this hour
    - queue adjustment
- **Peak Hours Prediction**
- **Volume Forecast** (e.g., next week)
- **Staffing Recommendations**
- **Key Predictive Insights**
- **Peak Patterns Analysis**

### Queue Performance
- Wait Time Trend
- Department Performance
- Performance Summary
  - overall avg wait
  - active departments
  - avg service time

### Volume & Throughput
- Service Distribution
- Ticket Volume breakdown
- Throughput Analysis
  - completion rate
  - active services
  - tickets/hour
  - estimated clear time

---

## 3) Departments Structure
Purpose: configure departments and services in a node-based tree view.

### Access
- Admin only

### Tree View
- Node-based tree structure
- Ability to create departments
- Ability to create services under a department

### Fields
- Department:
  - Name (Arabic)
  - Name (English)
- Service:
  - Name (Arabic)
  - Name (English)
  - Ticket prefix (set when creating the service)
  - Estimated wait time (configured value)

---

## 4) Organization
Purpose: organization/hospital metadata and user management.

### Access
- Admin only

### Organization Metadata
Editable fields:
- Logo
- Name
- Address
- Email
- Website

### Users & Access Control
Admin-only user management:
- Create / edit / disable users
- Assign role (Admin / IT / Manager / Staff)
- Assign **Manager department** (exactly one department per manager)
- Assign **Staff department** (exactly one department per staff user)
- Username is an **email address**
- Password policy (defaults)
  - Minimum length: **12**
  - Complexity: at least 1 uppercase, 1 lowercase, 1 digit, 1 symbol
  - Lockout: 5 failed attempts → 15 minute lock
- Password resets are performed by **Admin** (set temporary password + force change on next login)
- Optional: user profile picture (avatar)
  - Upload and storage handled by the backend (local disk + Docker volume) and served via authenticated API

---

## 5) User Experience
Purpose: manage end-user messaging and content.

### Responsibilities
- WhatsApp message templates (Arabic/English), per event
- Patient-facing text for kiosk and patient PWA (Arabic/English)

---

## 6) Mapping
Purpose: manage device mappings and operational bindings.

### Responsibilities
- Kiosk enrollment and configuration by Device ID (including remote config and apply-on-restart policy)
- Teller PC/counter binding by Device ID
- Counter-to-service binding (one service per counter)
- Signage player assignment to department zones
- (Optional) LED display adapter mappings if needed

---
