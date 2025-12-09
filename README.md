# Tiunelist
A collaborative life management platform.

## Tech Stack
**Frontend**: React, TypeScript
**Backend**: Rust (Axum framework)
**Data Source**: PostgreSQL, Redis
**Contracts**: Protobuf, OpenAPI specifications
**Authentication**: JWT with refresh tokens


## System Architecture
Tiunelist will operate as a Monorepo containing the Frontend, Backend and Shared Data Contracts.

### Key Engineering Decisions
Part of this project is for me to try out new technologies and application building approaches.
Some of the things I want to focus on here is:
- **Learning Rust**: as this is a new language for me, I want to practice building something real with it.
- **Using Event Driven Architecture**: User actions trigger events, that will get picked up by different relevant parts of the program to perform appropriate actions, for example save to the database, log the event, send a notification.
- **Two-tier Scheduler**: For something like notifications about pre-planned tasks I plan to implement a two-tier scheduler, where the tasks which are too far in the future are going to be stored in just the "cold" sql storage,
with an appropriate thaw process to move them into the event queue to be acted upon, but only when required not to overpopulate the queue.
- **Usage of protobuf**: I am interested to try protobuf for communication between the different application's parts, instead of a regular json, for example.
- **Content negotiation techniques**: Nonetheless, I am interested to also implement a content negotiation strategy where the backend would be able to return json, instead of protobuf stream if the request needs that.


## Feature Roadmap:
### Phase 1: Core Management & Security (MVP)
- **Workspaces & Projects**: Multi-tenant architecture allowing users to create isolated workspaces, manage members and organize work into projects.
- **Task Management**: Full lifecycle management CRUD, with status tracking, priorities, and rich-text descriptions.
- **User Permissions (RBAC)**: Granular role-based access control (Admin, Editor, Viewer) at both the Workspace and Project levels.
- **Notes System**: A comprehensive note-taking module supporting rich text, file,attachments and inter-linking between tasks and documents.
- **Audit & Logging System**: A persistent, immutable history of all actions taken within a workspace for compliance and recovery.

### Phase 2: Scheduling & Communication
- **Smart Notification System**: Multi-channel alerts (in-app, email, telegram)triggered by task assignments, due dates and mentions.
- **Calendar Synchronisation**: Two-way sync with external providers (Google Calendar, Outlook, Proton) to visualise tasks alongside schedules.
- **Sharing System**: Capabilities to share read-only views of projects or specific tasks with external users via secure public links.
- **Task Dependency System**: The ability to block a task until another task is complete, for more comprehensive planning.

### Phase 3: Governance & Insight
- **Activity Streams**: User-facing activity feeds showing recent updates in real-time.
- **Advanced Search**: Global search across tasks, notes, and file contents.

---

## Technical Decisions

### Core Stack (Finalized)
- **Database**: PostgreSQL
  - Superior multi-tenancy support with row-level security
  - JSONB for flexible rich-text storage
  - Native UUID support for security
  - Excellent full-text search capabilities
  
- **Web Framework**: Axum
  - Modern async-first design
  - Excellent type safety with extractors
  - Built on Tower middleware (compatible with tonic for gRPC)
  - First-class WebSocket support for real-time features
  
- **Database Access**: SQLx
  - Compile-time verification of SQL queries
  - Async-native for performance
  - Raw SQL control for learning and optimization
  - Built-in migration tooling

- **Authentication**: JWT with Refresh Tokens
  - Access tokens: Short-lived (15 min), stored in httpOnly cookies
  - Refresh tokens: Long-lived (7 days), stored in Redis
  - Password hashing: Argon2id
  - Future: Integration with external IdPs (Google, GitHub, etc.)

### Event Bus Architecture (Phased Approach)
**Design Principle**: Abstract event bus behind a trait for easy migration

**Phase 1**: In-memory event bus using tokio channels
- Fast to implement
- Sufficient for single-instance deployment
- No external dependencies

**Phase 2**: Redis Streams event bus
- Event persistence and replay capability
- Better reliability guarantees
- Supports multiple consumer groups

**Future**: RabbitMQ/Kafka if microservices architecture needed

---

## Implementation Plan

### Milestones
- M0: Repo hygiene and scaffolding (mono repo scripts, fmt/lint, CI smoke).
- M1: Auth and tenant bootstrap (signup/login, JWT+refresh, workspace creation, RBAC skeleton).
- M2: Tasks and notes MVP (CRUD, rich text, audit log, in-memory events).
- M3: Scheduling tier 1 (cold store + thaw job to queue).
- M4: Notifications and sharing (in-app/email/telegram adapters, public links).
- M5: Integrations and search (calendar sync, full-text search, activity streams).

### Backend (Axum + SQLx)
- Modules: auth, tenancy, tasks, notes, audit, events, notifications, scheduling.
- Auth: JWT access (15m) in httpOnly cookies, refresh in Redis (7d), Argon2id hashing, middleware for user/workspace context.
- Tenancy: `workspaces`, `workspace_members`, `projects`; RBAC roles (Admin/Editor/Viewer); RLS policies.
- Tasks/Notes: JSONB rich text, status/priority, attachments stub, links between tasks and notes; CRUD endpoints.
- Audit: append-only `audit_log` via event sink; actor/target/action/metadata JSONB.
- Events: trait-backed bus; in-memory tokio mpsc now, Redis Streams adapter later behind feature flag.
- Scheduling: `scheduled_notifications` table with `execute_at` and status; thaw job enqueues near-term items; idempotent consumer.
- Content negotiation: JSON first; protobuf behind `Accept: application/x-protobuf`.

### Frontend (React + TypeScript)
- Shell: workspace/project switcher, task list/detail, notes editor (TipTap/Slate), activity panel.
- Auth flows with refresh rotation and httpOnly cookies; graceful permission errors.
- Data: React Query; align models with shared contracts; start with JSON, protobuf later.
- Notifications: start with polling; WebSocket feed later.

### Shared Contracts
- Protobuf definitions for User/Workspace/Project/Task/Note/AuditEvent/Notification.
- OpenAPI for REST; generate TS types and Rust structs; document content-negotiation examples.

### Initial Data Model
- Users: id, email unique, password_hash, created_at.
- Workspaces: id, name, owner_id, created_at.
- Members: (workspace_id, user_id, role).
- Projects: id, workspace_id, name, description.
- Tasks: id, project_id, title, description_jsonb, status, priority, due_at, assignee_id, created_by.
- Notes: id, workspace_id, title, body_jsonb, linked_task_id?.
- Audit_log: id, workspace_id, actor_id, action, target_type/id, metadata_jsonb, created_at.
- Scheduled_notifications: id, workspace_id, task_id?, note_id?, execute_at, channel, payload_jsonb, status.

### First 6 Weeks (suggested)
- Week 1: Repo scaffolding, docker-compose (Postgres/Redis), migrations for users/workspaces/members/projects, Axum health check.
- Week 2: Auth (signup/login/refresh/logout), hashing, middleware, tests.
- Week 3: Workspace/project CRUD + RBAC guard for invites.
- Week 4: Task CRUD with rich-text JSONB; emit audit events via in-memory bus.
- Week 5: Notes CRUD, task links, audit-driven activity feed endpoint.
- Week 6: Scheduling table + thaw job; log-only notifier that consumes queue.

### Next Actions
- Add backend deps (`tracing`, `serde`, `thiserror`, `validator`, `sqlx` features) and create initial migrations.
- Scaffold Axum app state/router; wire Postgres/Redis configs from `.env`.
- Draft protobuf/OpenAPI skeletons for core entities.