# SubSpace

Lightweight web application for tracking recurring subscriptions, managing regular payments, and forecasting upcoming expenses.

🌐 **Live Demo:** [https://subbbspace.duckdns.org](https://subbbspace.duckdns.org)  
📖 **API Docs:** [https://subbbspace.duckdns.org/docs](https://subbbspace.duckdns.org/docs)

---

## Features

- **Subscription Tracking:** Add, edit, and track recurring payments with next billing dates.
- **Expense Forecasts:** Instant expense estimates for 30, 90, and 365 days ahead.
- **Authentication:** JWT tokens, bcrypt hashing, and email code verification via SMTP.
- **Automated Reminders:** Scheduled notifications sent before billing dates.
- **Observability:** Built-in Prometheus `/metrics` endpoint.

## Tech Stack

- **Backend:** FastAPI, Python 3.14, Uvicorn, SQLAlchemy (Async), asyncpg
- **Database:** PostgreSQL 15
- **Frontend:** Responsive Vanilla JS, HTML5, CSS (SPA)
- **Deployment:** Docker Compose, Caddy (Automatic HTTPS), GitHub Actions (CI/CD)

## Quick Start

1. **Clone & prepare config:**
   ```bash
   git clone https://github.com/mihail44b/subspace.git
   cd subspace
   cp .env.example .env
   ```
2. **Fill credentials in `.env`** (`POSTGRES_*`, `JWT_SECRET_KEY`, `SMTP_*`).
3. **Run with Docker Compose:**
   ```bash
   docker compose up -d --build
   ```

The app will be available at [http://localhost:8000](http://localhost:8000) (interactive docs at `/docs`).

## License

[MIT](LICENSE)
