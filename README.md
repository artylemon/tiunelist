# Tiunelist
A collaborative life management platform.

## Tech Stack
**Frontend**: React, TypeScript
**Backend**: Rust
**Data Source**: Relational database (not yet decided what db to use), Redis
**Contracts**: Protobuf, OpenAPI specifications

## System Architecture
Tiunelist will operate as a Monorepo containing the Frontend, Backend and Shared Data Contracts.

### Key Engineering Decisions
Part of this project is for me to try out new technologies and application building approaches.
Some of the things I want to focus on here is:
- **Learning Rust**: as this is a new language for me, I want to practice buildign something real with it.
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
- **Audit & Logging System**: A persistent, immutable history of all actions taken within a workspase for compliance and recovery.

### Phase 2: Scheduling & Communication
- **Smart Notification System**: Multi-channel alerts (in-app, email, telegram)triggered by task assignments, due dates and mentions.
- **Calendar Synchronisation**: Two-way sync with external providers (Google Calendar, Outlook, Proton) to visualise tasks alongside schedules.
- **Sharing System**: Capabilities to share read-only views of projects or specific tasks with external users via secure public links.
- **Task Dependency System**: The ability to block a task until another task is complete, for more comprehensive planing.

### Phase 3: Governance & Insight
- **Activity Streams**: User-facing activity feeds showing recent updates in real-time.
- **Advanced Search**: Global search across tasks, notes, and file contents.
