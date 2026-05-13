# ServiceDesk Pro — Problem Statement

> **Version:** 1.0  
> **Status:** In Progress  
> **Last Updated:** May 2026

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Background & Context](#2-background--context)
3. [Problem Analysis](#3-problem-analysis)
4. [Proposed Solution](#4-proposed-solution)
5. [Users & Roles](#5-users--roles)
6. [Core Workflows](#6-core-workflows)
7. [Functional Requirements](#7-functional-requirements)
8. [Non-Functional Requirements](#8-non-functional-requirements)
9. [System Architecture Overview](#9-system-architecture-overview)
10. [Technology Stack](#10-technology-stack)
11. [Project Phases](#11-project-phases)
12. [Out of Scope](#12-out-of-scope)
13. [Success Criteria](#13-success-criteria)

---

## 1. Executive Summary

Organizations of all sizes struggle with managing internal service requests, change approvals, and task routing across teams and departments. Emails get lost, approvals stall, accountability is unclear, and managers have no real-time visibility into team workload. Spreadsheets and ad-hoc chat tools create silos and leave no audit trail.

**ServiceDesk Pro** is an enterprise-grade internal ticketing and workflow management system that brings structure, accountability, and transparency to how work gets requested, reviewed, approved, and resolved — across multi-level organizational hierarchies.

It is purpose-built to mirror real-world tools like **Jira Service Management** and **ServiceNow**, constructed from the ground up as a full-stack portfolio project demonstrating:

- Clean Architecture principles
- Role-based security and authorization
- Formal workflow state machines
- Multi-level approval chains
- Real-time collaboration via SignalR
- SLA enforcement via background jobs
- Analytics and reporting dashboards
- AI-assisted ticket operations

---

## 2. Background & Context

In most organizations, internal service requests — whether for software access, infrastructure changes, budget approvals, or incident reports — travel through informal channels. A developer Slacks their manager. A manager forwards an email. Someone follows up in a standup. There is no system of record, no enforced process, and no visibility beyond the individuals directly involved.

As organizations grow, this breaks down in predictable ways:

- Work gets lost between handoffs
- Approvals happen outside any traceable process
- There is no way to measure how long things take
- Compliance and audit requirements go unmet
- Team leads have no data to manage workload fairly

Enterprise tools like ServiceNow and Jira Service Management solve this — but they are expensive, complex to configure, and opaque in how they work internally. ServiceDesk Pro solves the same problem from first principles, with a transparent, well-documented codebase that demonstrates exactly how such a system is built and why each design decision was made.

---

## 3. Problem Analysis

### 3.1 No Unified Request Management

When an employee needs something — a software access request, a production change, a bug escalation, or a procurement sign-off — there is no single place to raise it formally. Requests made via email, Slack, or verbally result in:

- No tracking or audit trail
- No defined ownership or accountability
- No SLA enforcement
- Requests falling through the cracks entirely

### 3.2 No Enforced Approval Chain

Different request types require different levels of authority to approve. A junior engineer cannot self-approve a production infrastructure change. An expenditure above a threshold requires finance sign-off. Without a system enforcing these chains:

- Approvals are bypassed or forgotten
- There is no record of who approved what and when
- Compliance and regulatory requirements cannot be demonstrated

### 3.3 No Supervisor Visibility

Team leads and managers have no real-time view of:

- What requests their team members have open
- Where bottlenecks exist in the approval pipeline
- Which team members are overloaded versus idle
- Whether SLAs are being met or missed

### 3.4 Work Stalls When Staff Are Absent

When the owner of a ticket goes on leave, work stops. There is no mechanism to:

- Proactively delegate ticket ownership before going on leave
- Allow supervisors to bulk-reassign work during unplanned absences
- Prevent tickets from becoming orphaned due to personnel availability

### 3.5 No Structured Feedback Loop

When a supervisor needs more detail before approving, the back-and-forth happens outside the system — in email or chat — losing all context and breaking the audit trail. There is no formal "return with comments" mechanism tied to the original request.

### 3.6 No Data-Driven Insights

Organizations cannot answer basic operational questions such as:

- How many tickets were resolved last month, broken down by team?
- What is the average approval cycle time end-to-end?
- Which ticket types are most frequently rejected or returned?
- Which supervisors are approval bottlenecks?

Without this data, process improvement is impossible and resource planning is guesswork.

### 3.7 Keyword-Dependent, Intent-Blind Search

Finding relevant past tickets requires knowing exact keywords used at the time of creation. There is no way to search by meaning or intent — for example, "find tickets similar to this one" or "show me all past database access requests regardless of how they were worded." Institutional knowledge locked in past tickets is effectively inaccessible.

---

## 4. Proposed Solution

ServiceDesk Pro is delivered in four distinct phases, each building on the last:

### Phase 1 — Backend API *(current)*
A production-grade ASP.NET Core Web API implementing clean architecture, role-based authorization, a formal ticket state machine, multi-level approval routing, real-time notifications via SignalR, SLA enforcement via Hangfire background jobs, file attachment handling via Azure Blob Storage, and structured audit logging across all operations.

### Phase 2 — React Frontend
A responsive, role-aware single-page application (SPA) in React + TypeScript. Each role receives a tailored experience — personal inbox, team workload view, approval action panels — driven entirely by the backend's authorization model. No hardcoded role logic in the UI.

### Phase 3 — Analytics & Reporting
Supervisor and admin dashboards surfacing ticket volume trends, resolution rates, SLA compliance percentages, team workload distribution, and approval cycle time analysis — visualized as interactive charts and exportable as CSV or PDF reports.

### Phase 4 — AI Layer
An intelligent assistance layer including:
- **Ticket Summarizer** — LLM-generated plain-English summary of a ticket and its full comment thread
- **Semantic Search** — vector embeddings enable search by intent, not just keywords
- **Auto-Categorizer** — classifies ticket type and suggests priority at submission time
- **Duplicate Detector** — cosine similarity on embeddings flags tickets already raised
- **Smart Priority Suggester** — recommends priority level based on description content

---

## 5. Users & Roles

| Role | Can Create | Can Assign | Can Submit | Approval Authority |
|---|---|---|---|---|
| **Individual Contributor** | ✅ | ✅ (pre-submission) | ✅ | None |
| **Supervisor L1** | ✅ | ✅ (within team/dept) | ✅ | Approves IC tickets |
| **Supervisor L2** | ✅ | ✅ (within dept) | ✅ | Auto-approved |
| **Admin** | ✅ | ✅ (anywhere) | ✅ | Auto-approved |

Each role sees only the menu options, data, and actions it is authorized for. Authorization is enforced at the API layer — the UI reflects it, but does not control it.

---

## 6. Core Workflows

### 6.1 Standard Ticket Lifecycle

### Submission Routing Rules
- IC submits → routes to IC's L1
- L1 submits → routes to L1's L2 (skips L1 review)
- L2 submits → auto-approved, audit logged
- Admin submits → auto-approved, audit logged

### Assignment Rules
- Any user can assign a ticket in Draft state
- L1/L2 can assign orphaned or unassigned tickets to ICs under them
- L1/L2 can reassign tickets when an IC leaves or is unavailable
- Once submitted, assignment is controlled by the workflow engine
- Supervisors can reassign within their scope during active workflow

### 6.2 Reassignment Workflow

```
Ticket assigned to Person A
    │
Person A goes on leave
    │
    ├── Option A: Person A pre-designated a delegate → auto-routed to Person B
    ├── Option B: L1 Supervisor manually reassigns individual ticket
    └── Option C: Admin or Supervisor performs bulk reassignment
                  (move all of Person A's open tickets to Person B in one action)

Person B receives ticket in inbox with full history, comments, and attachments intact
```

### 6.3 SLA & Escalation Workflow

```
Ticket enters a review state (L1 or L2)
    │
SLA timer starts (configurable per ticket type and review level, in business hours)
    │
    ├── At 75% of SLA window: warning notification sent to reviewer
    │
    └── At 100% (SLA breached):
            │
            ├── Escalation notification sent to next level supervisor
            └── If unresolved after escalation window: Admin is alerted
```

---

## 7. Functional Requirements

### 7.1 Authentication & Authorization
- JWT-based authentication with refresh token support
- Role-based access control enforced at the API layer on every endpoint
- Role-specific menu options and available actions in the UI
- Secure logout with token invalidation

### 7.2 Organization Hierarchy
- Admin creates and manages Departments and Teams
- Each Team has exactly one L1 Supervisor
- Each Department has exactly one L2 Supervisor
- Users belong to exactly one Team
- The hierarchy drives ticket routing, visibility scoping, and workload aggregation automatically — no manual configuration per ticket

### 7.3 Ticket Management
- ICs create tickets with: type, title, detailed description, priority, and optional file attachments
- Ticket types: `Change Request`, `Bug Report`, `Feature Request`, `Access Request`, `Incident`
- Priority levels: `Low`, `Medium`, `High`, `Critical`
- Tickets in `Draft` state are fully editable by the creator
- Submitted tickets are locked from editing unless formally returned by a reviewer
- All versions of a ticket's description are preserved when a ticket is revised and resubmitted

### 7.4 Workflow State Machine
- State transitions are validated and enforced server-side; no client request can force an invalid transition
- Every state change is permanently logged: actor, timestamp, from-state, to-state, optional comment
- Return actions require a mandatory comment explaining what additional detail is needed
- Approve and Reject actions support an optional comment
- The complete audit trail is viewable by all parties involved in the ticket's workflow

### 7.5 Inbox & Task Management
- Every user has a personal inbox showing tickets currently assigned to or awaiting action from them
- Inbox filters: status, priority, ticket type, date range
- Inbox sorting: date created, priority, SLA time remaining
- L1 Supervisors have a team view: all open tickets across their direct reports, sortable and filterable
- L2 Supervisors have a department view: all open tickets across all teams in their department

### 7.6 Comments & Collaboration
- Each ticket has a threaded, append-only comment section
- Comments are visible to all parties participating in the ticket's workflow
- When a supervisor returns a ticket, the return reason is posted as a system-generated comment alongside any personal comment
- Comment history is preserved permanently and cannot be edited or deleted

### 7.7 File Attachments
- File uploads supported at ticket creation time and during revision after a return
- Files stored in Azure Blob Storage; metadata (name, size, uploader, timestamp) stored in SQL Server
- All attachments from all versions of a ticket remain permanently accessible
- Attachment download requires authentication and role-appropriate ticket visibility

### 7.8 Notifications
- In-app notifications for: ticket assigned, ticket submitted, ticket returned, ticket approved, ticket rejected, ticket reassigned, SLA warning, SLA breach
- Email notifications for the same event set via SendGrid
- Per-user notification preferences: in-app only, email only, or both
- L1 and L2 supervisors receive a configurable daily digest of all pending items in their queue

### 7.9 Delegation & Absence Management
- A user can mark themselves as On Leave and optionally designate a delegate user
- While marked On Leave, new ticket assignments are automatically routed to the designated delegate
- L1 Supervisors can manually reassign any ticket within their team visibility scope
- Bulk reassignment: move all open tickets from one user to another in a single operation
- Ticket history records all reassignment events with actor and timestamp

### 7.10 Admin Panel
- Full user management: create, deactivate, assign roles, assign to teams
- Team and department management: create, update, set supervisors
- SLA rule configuration: per ticket type, per review level, expressed in business hours
- Escalation rule configuration: escalation threshold and escalation target per rule
- System-wide audit log: searchable, filterable log of all state changes and admin actions

---

## 8. Non-Functional Requirements

| Requirement | Target |
|---|---|
| **Security** | HTTPS only; JWT enforced on all protected routes; role checks at service layer, not just controller layer; no secrets in source control |
| **Auditability** | Every state change, comment, reassignment, and admin action permanently logged with actor identity and timestamp |
| **Scalability** | Stateless API design; horizontally scalable behind a load balancer; background jobs decoupled via Hangfire with SQL Server persistence |
| **Reliability** | Hangfire jobs survive application restart; failed jobs are retried with exponential backoff |
| **Performance** | Inbox API responds in under 300ms for datasets up to 10,000 tickets; database queries use appropriate indexing on foreign keys and status columns |
| **Observability** | Structured logging via Serilog with correlation IDs on all requests; log levels configurable per environment |
| **Testability** | Clean Architecture ensures domain and application logic are testable without infrastructure dependencies; integration tests cover all core workflow transitions |
| **Maintainability** | Each layer has a single, well-defined responsibility; no business logic in controllers; no direct database access outside the Infrastructure layer |

---

## 9. System Architecture Overview

ServiceDesk Pro follows **Clean Architecture** (also known as Onion Architecture), separating concerns into concentric layers where inner layers have no dependency on outer layers.

```
┌─────────────────────────────────────────────────────┐
│                    API Layer                        │
│         Controllers · SignalR Hubs · Middleware     │
├─────────────────────────────────────────────────────┤
│                Application Layer                    │
│       Use Cases · DTOs · Interfaces · Validators    │
├─────────────────────────────────────────────────────┤
│                  Domain Layer                       │
│    Entities · Enums · State Machine · Domain Events │
├─────────────────────────────────────────────────────┤
│               Infrastructure Layer                  │
│  EF Core · Repositories · Blob Storage · Email      │
│           Hangfire · SignalR · SendGrid             │
└─────────────────────────────────────────────────────┘
```

**Solution Structure:**

```
ServiceDesk/
├── ServiceDesk.API/              # Entry point: controllers, middleware, SignalR hubs, DI registration
├── ServiceDesk.Application/      # Use cases, command/query handlers, DTOs, service interfaces
├── ServiceDesk.Domain/           # Core entities, enums, value objects, state machine, domain events
├── ServiceDesk.Infrastructure/   # EF Core DbContext, repositories, Azure Blob, Hangfire, email, SignalR
└── ServiceDesk.Tests/            # Unit tests (domain/application) and integration tests (API)
```

---

## 10. Technology Stack

### Backend

| Concern | Technology |
|---|---|
| API Framework | ASP.NET Core Web API (.NET 9) |
| Authentication | ASP.NET Core Identity + JWT Bearer |
| ORM | Entity Framework Core (code-first migrations) |
| Database | SQL Server |
| Background Jobs | Hangfire (SLA timers, escalation, email digests) |
| Real-time | SignalR (live inbox updates) |
| File Storage | AWS S3 or Azure Blob Storage (TBD) |
| Email | SendGrid via MailKit |
| Logging | Serilog with structured output |
| API Docs | Swagger / OpenAPI (Swashbuckle) |
| Validation | FluentValidation |

### Frontend *(Phase 2)*

| Concern | Technology |
|---|---|
| Framework | React + TypeScript |
| State Management | Zustand or Redux Toolkit |
| UI Components | shadcn/ui |
| Charts | Recharts *(Phase 3)* |
| HTTP Client | Axios |
| Real-time | SignalR JS client |

### Infrastructure

| Concern | Technology |
|---|---|
| Cloud | Microsoft Azure |
| Hosting | Azure App Service |
| CI/CD | GitHub Actions |
| Containers | Docker (optional: AKS for Kubernetes deployment) |
| Secrets | Azure Key Vault / .NET User Secrets (local) |

### AI Layer *(Phase 4)*

| Concern | Technology |
|---|---|
| LLM | Anthropic Claude API or Azure OpenAI |
| Embeddings | Azure OpenAI text-embedding-ada-002 |
| Vector Search | Azure AI Search or pgvector |

---

## 11. Project Phases

### Phase 1 — Backend API
**Goal:** A fully functional, tested, documented REST API covering all core workflows.

Milestones:
- [ ] M1: Solution scaffold, EF Core, Identity, JWT auth
- [ ] M2: Org hierarchy (Departments, Teams, Users)
- [ ] M3: Ticket engine and state machine
- [ ] M4: Workflow routing and approval chain
- [ ] M5: Inbox, supervisor views, SignalR real-time
- [ ] M6: Comments, attachments, notifications
- [ ] M7: Admin panel, SLA rules, Swagger documentation

### Phase 2 — React Frontend
**Goal:** A role-aware SPA that fully consumes the backend API.

- Role-based layout and navigation
- IC ticket creation and resubmission flow
- Supervisor approval, return, and reject actions
- Personal inbox and team workload view
- Real-time inbox updates via SignalR

### Phase 3 — Analytics & Reporting
**Goal:** Data-driven dashboards for supervisors and admins.

- Monthly ticket volume by team and type
- Average approval cycle time trends
- SLA compliance rates
- Team workload distribution
- Export to CSV and PDF

### Phase 4 — AI Layer
**Goal:** Intelligent assistance embedded in the ticket workflow.

- Ticket summarizer (LLM-generated summary of ticket + thread)
- Semantic ticket search (vector embeddings, intent-based queries)
- Auto-categorizer and priority suggester on submission
- Duplicate detector across open and historical tickets

### Future Enhancements (Post Phase 4)
- Bulk ticket import from CSV or text file
  - Each row maps to a `CreateTicketCommand`
  - Same validation and routing logic as manual creation
  - Supported formats: CSV, JSON, plain text (TBD)

---

## 12. Out of Scope

The following are deliberately excluded from the current project roadmap:

- Mobile application (web-responsive only)
- Multi-tenant or multi-organization support
- Public-facing customer or client portal
- Payment, billing, or financial transaction workflows
- Native integration with external ITSM tools (ServiceNow, Jira, Zendesk)
- Custom drag-and-drop workflow builder

---

## 13. Success Criteria

### Phase 1 Complete When:
- [ ] All four roles authenticate and receive role-appropriate API access
- [ ] A ticket travels the full lifecycle: Draft → Submitted → L1 Review → L2 Review → Approved
- [ ] Return, reject, reassign, delegate, and bulk-reassign flows all function correctly
- [ ] Every state transition is audit-logged with actor identity, timestamp, and comment
- [ ] SLA timers start on ticket submission, fire warnings at 75%, and trigger escalation at 100%
- [ ] L1 Supervisors can view and act on their full team's open ticket workload
- [ ] L2 Supervisors can view the full department workload and return tickets to L1 or IC
- [ ] All endpoints are documented in Swagger with request/response schemas and example values
- [ ] Core workflow transitions are covered by integration tests

### Project Complete When:
- [ ] React frontend delivers a complete, role-aware user experience
- [ ] Analytics dashboards answer operational questions without manual data extraction
- [ ] AI features reduce time-to-triage and surface relevant historical context automatically

---

*ServiceDesk Pro is a portfolio project demonstrating enterprise-grade full-stack engineering. It is open source and built in public. Contributions, feedback, and questions are welcome.*