# Fi-Track 

> A headless full-stack personal finance application — CRUD for income, expenses, subscriptions, budgets, and investments with a dashboard overview, JWT authentication, and background email jobs via Celery.

---

## What is Fi-Track?

Fi-Track is a personal finance tracker that gives users full visibility over their financial life in one place. Users can log income and expenses, track active subscriptions (with auto-billing that rolls into expenses), set budgets, monitor investments, and view everything through a graphical dashboard — all behind a secure JWT-authenticated API.

---

## Architecture Overview

```
┌─────────────────────────────────┐
│        Vanilla JS Frontend      │
│     (static folder · no build)  │
└────────────────┬────────────────┘
                 │ HTTP
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                      FastAPI Backend                        │
│            JWT Auth · REST API · Business Logic             │
├──────────────┬──────────────────┬───────────────────────────┤
│   Finance    │   Subscription   │      Email Service        │
│   Module     │     Engine       │   (Celery + Redis)*       │
│ income/exp   │ auto-billing →   │  account verify ·         │
│ budget/inv   │ expense          │  password reset ·         │
│              │                  │  subscription reminders   │
└──────────────┴──────────────────┴───────────────────────────┘
                 │
                 ▼
        ┌─────────────────┐
        │  MySQL (TiDB)   │
        │  Cloud Database │
        └─────────────────┘
```

> \* Celery + Redis background job processing is fully implemented in the `main` branch. In the current self-hosted VPS deployment it is intentionally disabled to optimise container resource usage — the complete implementation remains available in the codebase.


## Features

### Authentication
- JWT-based authentication with access tokens
- Account verification via email (Celery background job)
- Password reset via email (Celery background job)

### Finance Modules
- **Income** — log and categorise income sources
- **Expenses** — track spending with category breakdown
- **Subscriptions** — manage recurring subscriptions; auto-billing triggers automatically add the charge as an expense entry and send a reminder email
- **Budgets** — set spending limits per category
- **Investments** — track investment entries and performance

### Dashboard
- Graphical overview of income vs expenses
- Budget utilisation at a glance
- Subscription and investment summaries

### Background Jobs (Celery)
- Email queue handled asynchronously so API responses are never blocked
- Jobs: account verification, password reset, subscription reminders, auto-billing notifications
- Redis used as the message broker for the Celery worker

### Logging
- Request and error logging implemented across the application for observability

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend framework | FastAPI (Python) |
| Frontend | Vanilla JavaScript |
| Database | MySQL via TiDB Cloud |
| Background jobs | Celery + Redis *(main branch)* |
| Authentication | JWT |
| Email service | SMTP via Celery worker |
| Deployment | Self-hosted VPS · Coolify |

---

## Key Design Decisions

**Why FastAPI?**
FastAPI's async support and automatic OpenAPI docs made it a natural fit for a REST API with multiple resource types. Pydantic schemas enforce request validation at the boundary, keeping business logic clean.

**Why a headless architecture?**
Separating the backend (`api/`) from the frontend (`static/`) keeps the API independently usable — the same endpoints could power a mobile app or a third-party integration without any changes.

**Why Vanilla JS for the frontend?**
The goal was to keep the frontend lightweight and dependency-free. No build step, no bundler — just static files served directly, which also makes deployment simpler.

**Why Celery for emails?**
Sending emails synchronously inside an API request blocks the response and creates a poor user experience. Celery offloads email delivery to a background worker so the API responds immediately and the email is delivered asynchronously.

**Why is Celery disabled in production?**
On a self-hosted VPS with constrained resources, running a Celery worker container alongside Redis adds significant memory overhead. The decision was made to disable background jobs in the deployed branch to keep all other services stable — a deliberate trade-off between features and resource constraints. The full implementation is available in the `main` branch.

---

## Branch Structure

| Branch | Description |
|---|---|
| `main` | Full codebase including Celery + Redis email jobs |
| `deploy-branch` | Production branch — Celery disabled for VPS resource optimisation |

---

[![Live Demo](https://img.shields.io/badge/Live%20Demo-Click%20to%20Explore-16A34A?style=for-the-badge)](http://fi-track.manavkashyap.com)

## Author

**Abhinav** · [GitHub @Abhinav-IN](https://github.com/Abhinav-IN)
