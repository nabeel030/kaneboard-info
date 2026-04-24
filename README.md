<p align="center">
  <img src="assets/logo.svg" alt="Kaneboard Logo" width="96" />
</p>

<h1 align="center">Kaneboard — Project Management for Modern Teams</h1>

<p align="center">
  <a href="https://kaneboard.com"><img src="https://img.shields.io/badge/kaneboard.com-live-4ade80?style=flat-square&logo=google-chrome&logoColor=white" alt="Website"></a>
  <a href="https://laravel.com"><img src="https://img.shields.io/badge/Laravel-12.x-FF2D20?style=flat-square&logo=laravel&logoColor=white" alt="Laravel"></a>
  <a href="https://vuejs.org"><img src="https://img.shields.io/badge/Vue.js-3.x-42b883?style=flat-square&logo=vue.js&logoColor=white" alt="Vue.js"></a>
  <a href="https://php.net"><img src="https://img.shields.io/badge/PHP-8.2+-777BB4?style=flat-square&logo=php&logoColor=white" alt="PHP"></a>
  <img src="https://img.shields.io/badge/License-Proprietary-red?style=flat-square" alt="License">
</p>

> **Kaneboard** is a web-based project management platform built on the [Kanban](https://en.wikipedia.org/wiki/Kanban_(development)) methodology, available at [kaneboard.com](https://kaneboard.com). It provides software development teams with a unified workspace to plan, track, and deliver work — powered by Laravel 12, Vue 3, and an integrated AI assistant called KaneBot.

---

## Table of Contents

- [Overview](#overview)
- [History](#history)
- [Features](#features)
  - [Workspace Management](#workspace-management)
  - [Project Management](#project-management)
  - [Kanban Board](#kanban-board)
  - [Ticket Management](#ticket-management)
  - [Time Tracking](#time-tracking)
  - [Daily Standup Module](#daily-standup-module)
  - [KaneBot — AI Assistant](#kanebot--ai-assistant)
  - [Calendar View](#calendar-view)
  - [Team Messaging](#team-messaging)
  - [Notifications](#notifications)
  - [Role-Based Access Control](#role-based-access-control)
  - [Dashboard & Analytics](#dashboard--analytics)
  - [Authentication](#authentication)
- [Technical Architecture](#technical-architecture)
- [Technology Stack](#technology-stack)
- [License](#license)
- [External Links](#external-links)
- [See Also](#see-also)

---

## Overview

Kaneboard is a self-hosted project management platform designed around the **Kanban** workflow model. It enables teams to organize their work into *workspaces*, *projects*, and *tickets*, and to visualize the progress of those tickets across a six-column Kanban board.

Beyond basic task tracking, Kaneboard includes:

- An integrated AI assistant (**KaneBot**) for project queries and ticket drafting
- Automated standup report generation from ticket activity
- Real-time time tracking with live timers
- Project health scoring with risk signal detection
- AI-powered ticket description generation and CSV ticket imports
- Comprehensive role-based access control (RBAC) via Spatie Laravel Permission

The application is proprietary and self-hostable on any PHP-compatible server environment.

---

## History

Kaneboard began as a simple, clean Kanban board built on the official Laravel Vue Starter Kit — a straightforward task management tool supporting basic ticket and project CRUD operations. Over successive development iterations, it evolved into a full-featured project management platform with:

- Multi-workspace support and role-based access control
- AI integration via the Laravel AI package and the KaneBot assistant
- Time logging and live timer support
- Daily standup submission and team timeline views
- Project health scoring and risk signal detection
- Firebase-powered push notifications and social login
- AI-driven CSV/document ticket imports
- Ticket revision history tracking
- Blog and landing page modules for public-facing content

---

## Features

### Workspace Management

Kaneboard organizes all work inside isolated **Workspaces**. Each workspace is owned by a single user and can contain unlimited members and projects.

- Create, rename, and delete workspaces
- Invite and manage workspace members
- Switch between multiple workspaces without logging out
- Set a default workspace for automatic session resolution on login
- Workspace-scoped roles and permissions (Owner, custom roles)
- Workspace-scoped Spatie permission caching for performance

---

### Project Management

Within a workspace, users organize work into **Projects**. Projects support:

- Full CRUD with permission guards
- Project start date, end date, and **baseline schedule** tracking for schedule variance analysis
- Project file attachments (images, PDFs, ZIP archives, office documents — up to 10 MB per file)
- Per-member time tracking aggregation showing tracked hours per contributor
- **Project Health Scoring** via the `ProjectHealthService`, which calculates:
  - `ON_TRACK`, `AT_RISK`, `LATE`, `COMPLETED`, `NO_SCHEDULE`, or `INVALID_SCHEDULE` statuses
  - Expected vs. actual progress percentage
  - Forecasted end date and confidence score
  - Risk signals (e.g., schedule overrun, no recent activity)
- AI-driven ticket imports from uploaded documents (CSV, text files)
- Project membership management (add/remove members, manage roles within the project)

---

### Kanban Board

The Kanban board is the central workspace for ticket flow management.

- **Six workflow columns**: `Backlog → Todo → In Progress → Done → Tested → Completed`
- **Drag-and-drop** reordering of tickets within and across columns (powered by `vuedraggable`)
- Ticket position persisted to the database on every drag event
- Project selector dropdown to switch board view between projects
- Visual color-coded priority and type badges on ticket cards
- Overdue ticket highlighting for tickets past their deadline
- Quick-add ticket modal directly within any column

---

### Ticket Management

Tickets are the atomic unit of work in Kaneboard. Each ticket supports a rich set of attributes:

| Attribute | Details |
|---|---|
| **Title** | Required, up to 120 characters |
| **Description** | Optional, up to 2,000 characters |
| **Status** | `backlog`, `todo`, `in_progress`, `done`, `tested`, `completed` |
| **Type** | Feature, Bug, Improvement, Task, or custom enum values |
| **Priority** | `low`, `medium`, or `high` |
| **Assignee** | Any workspace member who is also a project member |
| **Deadline** | Date-based; triggers overdue detection |
| **Attachments** | Images, PDFs, ZIP files — up to 10 MB each |

Additional ticket capabilities:

- **Auto `started_at` timestamp** — set when a ticket first moves to `in_progress`
- **Auto `completed_at` timestamp** — set when a ticket moves to a done state; cleared if moved back
- **Overdue detection** — `is_overdue` computed attribute when deadline is past and ticket is not complete
- **AI Description Generation** — the `TicketAgent` writes or refines a professional description based on the ticket title and type
- **Ticket comments** — threaded discussion per ticket with author attribution
- **Revision History** — every field change is recorded in `ticket_revisions` via an Eloquent model observer, capturing old value, new value, field name, and the user who made the change

---

### Time Tracking

Built-in time tracking at the ticket level:

- **Live Timer** — start and stop a timer directly on a ticket detail page with real-time elapsed display
- **Time Logs** — every timer session stored as a `TicketTimeLog` record with `started_at`, `ended_at`, and `duration_seconds`
- **Automatic timer stop** — active timers are ended when a ticket moves to a done state
- **Aggregated tracking** — `tracked_seconds` and `tracked_hours` computed via an optimized subquery (`scopeWithTrackedSeconds`)
- **Per-member time breakdown** on the Project page, ranking contributors by total hours logged
- **Total project hours** surfaced in both the project view and the dashboard

---

### Daily Standup Module

Async daily standups for distributed teams:

- **My Standup form** — submit three fields per workday: *What I did yesterday*, *What I'll do today*, *Blockers*
- **Auto-generate from ticket activity** — one click populates the standup form from the user's actual completed and in-progress tickets
- **AI-generated flag** — auto-generated submissions are marked `is_ai_generated` for transparency
- **Team board** — all workspace members with their standup status for today (submitted / pending), sorted so submitted members appear first
- **Standup history** — paginated list of the current user's past standups
- **Member timeline** — calendar-style view of a member's standup submissions over the last 7, 14, or 30 days, with weekends labeled and workday gaps highlighted
- **Owner notifications** — workspace owners receive a system notification when a member submits a standup

---

### KaneBot — AI Assistant

**KaneBot** is Kaneboard's integrated AI assistant, gated behind the `kanebot.agent` permission. Accessible at `/kanebot`, it operates in two modes:

- **Full KaneBotAgent** — has access to structured workspace context (projects, tickets, members, health scores) and can answer questions about the current state of your work
- **KaneBotLiteAgent** — a lightweight mode for general queries without workspace context injection

KaneBot capabilities include answering questions about project health, overdue tickets, team activity, and generating AI-assisted ticket descriptions.

---

### Calendar View

- Visual calendar displaying tickets by their deadline date
- Gated behind the `calendar.view` permission
- Provides a time-based overview of upcoming work and overdue items across all projects in a workspace

---

### Team Messaging

- Real-time team messaging within a workspace, gated behind the `messaging` permission
- Enables team communication without leaving the project management context

---

### Notifications

- In-app notification system with a bell icon and unread count badge
- `SystemActivityNotification` class for workspace-level events (e.g., standup submissions)
- **Mark as read / Mark all as read** actions with type-safe route helpers via Wayfinder

---

### Role-Based Access Control

Kaneboard implements a granular, workspace-scoped RBAC system using **Spatie Laravel Permission** (`^7.2`):

- Roles are **workspace-scoped** — each workspace has its own role set
- The workspace creator is automatically assigned the `Owner` role with all permissions
- Custom roles can be created by workspace owners with any combination of permissions

| Permission Group | Permissions |
|---|---|
| Workspaces | `workspaces.view`, `workspaces.manage` |
| Members | `members.view`, `members.create`, `members.update`, `members.delete`, `members.manage` |
| Projects | `projects.view`, `projects.create`, `projects.update`, `projects.delete` |
| Tickets | `tickets.view`, `tickets.create`, `tickets.update`, `tickets.delete`, `tickets.assign` |
| Time Logs | `timelogs.view`, `timelogs.manage` |
| Board | `board.view`, `board.reorder` |
| Roles | `roles.view`, `roles.create`, `roles.update`, `roles.delete` |
| Features | `kanebot.agent`, `messaging`, `calendar.view`, `standup.view` |

All controllers use `permission:` middleware guards and/or Laravel Policies for resource-level authorization.

---

### Dashboard & Analytics

The Dashboard renders a rich, role-aware analytics view.

**For all users (KPI cards):**
- Total projects, total tickets, open tickets, in-progress tickets
- Overdue count, due-soon count (within 3 days), tickets completed in the last 7 days
- My assigned open tickets and my in-progress tickets

**For workspace owners (additional panels):**
- **Project Health Summary** — count of projects per health state, cached for 5 minutes
- **Team Activity Table** — per-member breakdown of open, completed, and overdue tickets for up to 15 members, cached for 3 minutes
- **30-day Velocity Trend** chart — daily completed ticket counts for the past 30 days
- **Risky Projects Panel** — up to 6 projects flagged as Late or At Risk, sorted by severity
- **Member Count** for the current workspace

**For non-owner members:**
- **My Tickets by Priority** — breakdown of assigned open tickets by priority
- **Recent Activity** — last 5 tickets the user created or was assigned to

**Shared analytics panels:**
- 14-day Completion Trend chart
- Status Distribution chart (ticket count per Kanban column)
- Risk Tickets table (overdue and due-soon tickets with project context)

---

### Authentication

- **Email/password** authentication via **Laravel Fortify** (`^1.30`)
- **Email verification** required for access to protected routes
- **Two-factor authentication (2FA)** support
- **Social login** via **Laravel Socialite** (`^5.26`) — supports OAuth providers such as Google
- **Firebase Authentication** for mobile or Firebase-based login flows

---

## Technical Architecture

Kaneboard follows a **monolithic MVC architecture** enhanced by Inertia.js for SPA-like navigation.

```
┌─────────────────────────────────────────┐
│           Frontend (Vue 3 + TS)         │
│  Pages, Components, Composables,        │
│  Pinia Stores, Wayfinder Routes         │
├─────────────────────────────────────────┤
│        Inertia.js Bridge Layer          │
│  Server-side rendering (SSR) supported  │
├─────────────────────────────────────────┤
│      Laravel Backend (PHP 8.2+)         │
│  Controllers, Models, Policies,         │
│  Services, Jobs, Notifications,         │
│  Observers, AI Agents, Enums            │
├─────────────────────────────────────────┤
│         Database (MySQL)                │
│  34 migrations covering all entities    │
└─────────────────────────────────────────┘
```

**Key architectural patterns:**

- **Eloquent Model Observers** — `TicketRevision` tracking via `static::updated()` hooks, cleanly separating audit logic from controllers
- **Service Layer** — `ProjectHealthService`, `AiProviderService`, `BadgeService` encapsulate complex business logic outside of controllers
- **Laravel AI Agents** — `KaneBotAgent`, `KaneBotLiteAgent`, and `TicketAgent` as dedicated agent classes for clean AI prompt composition
- **Wayfinder** — generates type-safe TypeScript route helpers from Laravel route definitions, eliminating string-based URL construction on the frontend
- **Workspace-scoped Spatie permissions** for multi-tenant RBAC
- **Database-level caching** via `Cache::remember()` on expensive dashboard aggregations
- **Queue-based background processing** for jobs and notifications
- **SSR support** via `ssr.ts` entry point and `php artisan inertia:start-ssr`

---

## Technology Stack

| Layer | Technology | Version |
|---|---|---|
| **Backend Framework** | Laravel | ^12.0 |
| **PHP** | PHP | ^8.2 |
| **Frontend Framework** | Vue 3 (Composition API + TypeScript) | Latest |
| **SPA Bridge** | Inertia.js | ^2.0 |
| **Styling** | Tailwind CSS | Latest |
| **Authentication** | Laravel Fortify | ^1.30 |
| **Social Auth** | Laravel Socialite | ^5.26 |
| **Permissions** | Spatie Laravel Permission | ^7.2 |
| **AI Integration** | Laravel AI | ^0.4.1 |
| **Push Notifications** | Firebase PHP SDK (kreait) | ^8.2 |
| **Type-safe Routing** | Laravel Wayfinder | ^0.1.9 |
| **Ziggy (JS Routes)** | Tightenco Ziggy | ^2.6 |
| **Drag & Drop** | vuedraggable | — |
| **SEO / Sitemap** | Spatie Laravel Sitemap | ^8.0 |
| **Code Style** | Laravel Pint | ^1.24 |
| **Testing** | PHPUnit | ^11.5 |

---

## License

Kaneboard is proprietary software. All rights are reserved by the developer. Use, modification, and redistribution are subject to the terms of the Kaneboard commercial license agreement. Unauthorized copying or distribution of the software is prohibited.

---

## External Links

- 🌐 [kaneboard.com](https://kaneboard.com) — Official Website

---

## See Also

- [Kanban (development) — Wikipedia](https://en.wikipedia.org/wiki/Kanban_(development))
- [Laravel](https://en.wikipedia.org/wiki/Laravel)
- [Vue.js](https://en.wikipedia.org/wiki/Vue.js)
- [Inertia.js](https://inertiajs.com/)
- [Scrum (software development)](https://en.wikipedia.org/wiki/Scrum_(software_development))
- [Jira (software)](https://en.wikipedia.org/wiki/Jira_(software))
- [Trello](https://en.wikipedia.org/wiki/Trello)

---

*This repository serves as the public reference documentation for [Kaneboard](https://kaneboard.com). For the traditional Japanese Kanban manufacturing system, see [Kanban](https://en.wikipedia.org/wiki/Kanban).*