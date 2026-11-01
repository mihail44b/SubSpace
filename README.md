# SubSpace

Web application for tracking recurring subscriptions and payments.

This checkout contains application source code. Local environment setup,
containerization, deployment, backups and monitoring will be built step by step.

## Application

- Backend: Python, FastAPI, Uvicorn.
- Database: PostgreSQL, SQLAlchemy async, asyncpg.
- Frontend: HTML, CSS and JavaScript in `app/static/`.
- Authentication: JWT and bcrypt.
- Email verification and payment reminders: SMTP.

## Source layout

- `app/main.py`: application entry point (`app.main:app`).
- `app/config.py`: environment-based settings.
- `app/database.py`: database connections and initial table creation.
- `app/models.py`, `app/schemas.py`: data models and API schemas.
- `app/routes.py`, `app/crud.py`: API routes and database operations.
- `app/auth.py`, `app/email.py`: authentication and email helpers.
- `app/static/`: frontend assets.
- `scripts/notify_cron.py`: payment reminder application script.
- `requirements.txt`: Python dependencies.

## Configuration to prepare

Database connection: `DATABASE_URL`.
Authentication: `JWT_SECRET_KEY`, `JWT_ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES`.
Email: `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `SMTP_FROM`.

Runtime settings and credentials are not included. Startup instructions will be
added after the local setup is verified.
