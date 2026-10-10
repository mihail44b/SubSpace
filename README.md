# SubSpace — Subscription Manager

A lightweight, self-hosted web application for tracking recurring subscriptions and payments. Built with **FastAPI**, **PostgreSQL**, and a vanilla JS frontend, it gives you a clear picture of where your money goes — and where it's about to go.

Live instance: [subbbspace.duckdns.org](https://subbbspace.duckdns.org)

---

## Features

- **Subscription Tracking** — Add, edit, activate/deactivate, and delete subscriptions with monthly or yearly billing periods.
- **One-Click Payment** — Mark a subscription as paid; the next payment date automatically shifts forward by the billing period.
- **Categories** — Organize subscriptions into categories (5 built-in defaults: *Streaming, Utilities, Mobile, Software, Services*) or create your own.
- **Spending Analytics** — View current-month spend broken down by category and forecasts for 30, 90, and 365 days ahead (powered by a recursive SQL CTE).
- **Multi-Currency** — Each user selects their preferred currency (3-letter ISO code).
- **Email Verification** — 6-digit code sent on registration; accounts must be verified before login.
- **Password Reset** — Secure code-based password reset flow via email.
- **Payment Reminders** — A cron script (`scripts/notify_cron.py`) emails users 24 hours before their subscriptions are due.
- **JWT Authentication** — Stateless Bearer token auth with configurable expiry.
- **SPA Frontend** — Single-page application served from `app/static/`; all non-API routes fall through to `index.html`.
- **Docker-Ready** — Ship with `docker compose up` in seconds.
- **CI/CD** — GitHub Actions workflow auto-deploys to a server via SSH on every push to `main`.

---

## Tech Stack

| Layer       | Technology                                       |
|-------------|--------------------------------------------------|
| Backend     | Python 3.11, FastAPI, Uvicorn                    |
| ORM         | SQLAlchemy 2.0 (async) + asyncpg                 |
| Database    | PostgreSQL 15                                    |
| Auth        | JWT (PyJWT) + bcrypt                             |
| Validation  | Pydantic v2, pydantic-settings, email-validator  |
| Email       | SMTP (smtplib) via anyio thread offloading       |
| Frontend    | Vanilla HTML / CSS / JS (SPA)                    |
| Deployment  | Docker, Docker Compose, Caddy (reverse proxy)    |
| CI/CD       | GitHub Actions → SSH deploy                      |

---

## Project Structure

```
SubscribeManager/
├── app/
│   ├── main.py          # FastAPI app, lifespan, CORS, static mount, SPA catch-all
│   ├── config.py         # Pydantic Settings (DB, JWT, SMTP) loaded from .env
│   ├── database.py       # Async engine, session factory, init_db (table creation + seed)
│   ├── models.py         # SQLAlchemy models: User, VerificationCode, Category, Subscription, PaymentHistory
│   ├── schemas.py        # Pydantic request/response schemas + analytics DTOs
│   ├── auth.py           # Password hashing, JWT creation/verification, get_current_user dependency
│   ├── crud.py           # All database operations (users, categories, subscriptions, analytics)
│   ├── routes.py         # API router: auth, categories, subscriptions, analytics endpoints
│   ├── email.py          # SMTP email helpers (verification, reset, payment reminders)
│   └── static/           # Frontend SPA (index.html, css/, js/, icons/)
├── scripts/
│   └── notify_cron.py    # Standalone cron script: emails users about upcoming payments
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env                  # (gitignored) environment variables
└── .github/workflows/
    └── deploy.yml        # Auto-deploy on push to main
```

---

## Getting Started

### Prerequisites

- **Docker** & **Docker Compose** (recommended), or
- **Python 3.11+** and a running **PostgreSQL 15** instance

### 1. Clone the Repository

```bash
git clone https://github.com/mihail44b/SubscribeManager.git
cd SubscribeManager
```

### 2. Create a `.env` File

```dotenv
DATABASE_URL=postgresql+asyncpg://subspace_user:SecretPostgresPassword123!@db:5432/subspace_db
JWT_SECRET_KEY=change-me-to-a-long-random-string
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password
SMTP_FROM=your-email@gmail.com
```

> [!IMPORTANT]
> For Gmail, use an [App Password](https://support.google.com/accounts/answer/185833) — not your regular password.

### 3. Run with Docker Compose

```bash
docker compose up -d --build
```

This starts two services:

| Service | Container        | Description                          |
|---------|------------------|--------------------------------------|
| `db`    | `subspace_db`    | PostgreSQL 15 with a health check    |
| `web`   | `subspace_web`   | FastAPI app on `127.0.0.1:8000`      |

The app auto-creates all tables and seeds default categories on first startup.

### 4. Access the App

- **Frontend**: http://localhost:8000
- **API docs** (Swagger): http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

---

## Running Without Docker

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Make sure PostgreSQL is running and DATABASE_URL in .env points to it
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

---

## API Overview

All endpoints are prefixed with `/api`.

### Authentication

| Method | Endpoint                      | Description                        | Auth  |
|--------|-------------------------------|------------------------------------|-------|
| POST   | `/api/auth/register`          | Create account, sends verification | No    |
| POST   | `/api/auth/verify-email`      | Submit 6-digit verification code   | No    |
| POST   | `/api/auth/resend-verification` | Re-send verification code        | No    |
| POST   | `/api/auth/login`             | Get JWT access token               | No    |
| POST   | `/api/auth/forgot-password`   | Request password reset code        | No    |
| POST   | `/api/auth/reset-password`    | Reset password with code           | No    |
| GET    | `/api/auth/me`                | Get current user profile           | Yes   |
| PATCH  | `/api/auth/me`                | Update user currency               | Yes   |

### Subscriptions

| Method | Endpoint                           | Description                                  | Auth |
|--------|------------------------------------|----------------------------------------------|------|
| GET    | `/api/subscriptions`               | List all user subscriptions                  | Yes  |
| POST   | `/api/subscriptions`               | Create a new subscription                    | Yes  |
| GET    | `/api/subscriptions/{id}`          | Get a single subscription                    | Yes  |
| PUT    | `/api/subscriptions/{id}`          | Update a subscription                        | Yes  |
| DELETE | `/api/subscriptions/{id}`          | Delete a subscription                        | Yes  |
| POST   | `/api/subscriptions/{id}/pay`      | Record payment & advance next payment date   | Yes  |

### Categories

| Method | Endpoint            | Description                     | Auth |
|--------|---------------------|---------------------------------|------|
| GET    | `/api/categories`   | List default + user categories  | Yes  |
| POST   | `/api/categories`   | Create a custom category        | Yes  |

### Analytics

| Method | Endpoint          | Description                                           | Auth |
|--------|-------------------|-------------------------------------------------------|------|
| GET    | `/api/analytics`  | Current-month spend by category + 30/90/365 day forecast | Yes |

---

## Payment Reminder Cron

The script `scripts/notify_cron.py` queries for active subscriptions due **tomorrow** and sends grouped email notifications to verified users.

Run it manually:

```bash
python scripts/notify_cron.py
```

Or schedule it with cron (e.g., daily at 09:00):

```cron
0 9 * * * cd /path/to/SubscribeManager && /path/to/.venv/bin/python scripts/notify_cron.py
```

---

## Deployment (CI/CD)

The included GitHub Actions workflow (`.github/workflows/deploy.yml`) triggers on every push to `main`:

1. SSHes into the production server
2. Pulls the latest code
3. Rebuilds and restarts Docker containers
4. Reloads Caddy reverse proxy

Required GitHub Secrets: `SSH_HOST`, `SSH_USER`, `SSH_KEY`.

---

## Environment Variables

| Variable                     | Default                              | Description                          |
|------------------------------|--------------------------------------|--------------------------------------|
| `DATABASE_URL`               | `postgresql+asyncpg://…`             | Async PostgreSQL connection string   |
| `JWT_SECRET_KEY`             | *(change in production!)*            | Secret for signing JWT tokens        |
| `JWT_ALGORITHM`              | `HS256`                              | JWT signing algorithm                |
| `ACCESS_TOKEN_EXPIRE_MINUTES`| `1440` (24h)                         | Token lifetime in minutes            |
| `SMTP_HOST`                  | `smtp.gmail.com`                     | SMTP server hostname                 |
| `SMTP_PORT`                  | `587`                                | SMTP server port (TLS)               |
| `SMTP_USER`                  | —                                    | SMTP login username                  |
| `SMTP_PASSWORD`              | —                                    | SMTP login password / app password   |
| `SMTP_FROM`                  | —                                    | "From" address for outgoing emails   |

---

## License

This project is licensed under the **GNU General Public License v3.0** — see the [LICENSE](LICENSE) file for details.
