# Rex VMS Backend

Flask + PostgreSQL REST API for **Rex VMS**, a visitor management system modeled on the reception and security workflow at Rex Insurance. It was built as a SIWES project during my placement there.

**Live app:** https://rex-vms.vercel.app/
**Frontend repo:** https://github.com/Luthur24/rex-vms

---

## What it does

The API serves three role-specific portals, each protected by its own passcode:

| Portal | Who uses it | What it does |
|---|---|---|
| **Security** | Gate staff | Register visitors, search and edit visitor records, log a visit (department, host, purpose) |
| **Front Desk** | Reception | Run the live visitor status board: approve, hold or reject visitors, mark them with their host, check them out |
| **Admin** | Management | Dashboard stats, full visit history, staff roster, active sessions, login history, passcode changes |

There is also a short **AI help assistant** (Mistral) that answers questions about whatever the current portal can actually do.

---

## Visit lifecycle

Front Desk can only move a visit along valid steps. Any other change is rejected with a `400`.

```
pending ──► approved ──► with-host ──► checked-out
   │
   ├──► hold ──► approved / rejected (or stay on hold)
   │
   └──► rejected
```

---

## Authentication and sessions

- Login takes a **portal**, a **name** and a **passcode**.
- Portal passcodes are stored as **hashed** values (Werkzeug), never in plain text.
- A successful login creates a random session token (`secrets.token_urlsafe`) stored in the database and returns it to the client.
- The client sends it on every request as `Authorization: Bearer <token>`.
- Each endpoint checks that the token is active **and belongs to the correct portal**. A Security token cannot call a Front Desk endpoint.
- `last_seen` is updated on each authenticated request, so the Admin portal can see who is active.
- Every login is recorded in a login history table.
- Passcodes are seeded from environment variables **only on first run**. After that they are managed from the Admin portal.

---

## API reference

### General
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/ping` | Health check and cold-start warm-up |

### Auth
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/login` | Log in with `portal`, `name`, `passcode` |
| POST | `/api/auth/logout` | End the current session |
| GET | `/api/auth/verify` | Check whether a token is still valid |

### Security portal
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/visitors?search=` | List visitors, optionally searching by name, phone or ID number |
| POST | `/api/visitors` | Register a visitor |
| PUT | `/api/visitors/<id>` | Edit a visitor's details |
| GET | `/api/visitors/<id>/visits` | A visitor's visit history |
| POST | `/api/visitors/<id>/visits` | Log a new visit (starts as `pending`) |

Visitor validation: name of at least 2 characters, an 11-digit phone number, an ID type and an ID number of at least 3 characters.

### Front Desk portal
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/visits` | Today's visits for the live status board |
| PATCH | `/api/visits/<id>` | Move a visit to its next valid status |

### Admin portal
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/admin/stats` | Visitors today and this month, currently inside, pending approvals, top hosts (last 7 days), recent activity |
| GET | `/api/admin/visits?search=&limit=&offset=` | Full visit history with search and paging (max 100 per page) |
| GET | `/api/admin/roster` | List staff roster |
| POST | `/api/admin/roster` | Add a staff member |
| PATCH | `/api/admin/roster/<id>` | Toggle a staff member between active and suspended |
| GET | `/api/admin/sessions` | Currently active sessions |
| GET | `/api/admin/login-history` | Login history (max 200 entries) |
| POST | `/api/admin/passcode` | Change a portal's passcode (minimum 4 characters) |

> The staff roster is a directory only. It is not connected to portal login.

### AI assistant
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/ai/ask` | Ask a question for a given portal. The model is given that portal's real feature list so answers stay grounded. |

---

## Database

Tables are created automatically on startup with `CREATE TABLE IF NOT EXISTS`, so existing data is never wiped.

| Table | Purpose |
|---|---|
| `rexvms_portal_passcodes` | Hashed passcode per portal |
| `rexvms_sessions` | Session tokens, activity and active flag |
| `rexvms_login_history` | Login and logout times |
| `rexvms_staff_roster` | Staff directory with roles and status |
| `rexvms_visitors` | Registered visitors and ID details |
| `rexvms_visits` | Visits linked to visitors, with host, department, status, check-in and check-out |

---

## Tech stack

- Python, Flask, Flask-CORS
- PostgreSQL via `psycopg2`
- Werkzeug password hashing
- Mistral API (via `requests`)
- Deployed on Render with Gunicorn; frontend on Vercel

---

## Running locally

```bash
git clone https://github.com/Luthur24/rex-vms-backend.git
cd rex-vms-backend
pip install -r requirements.txt
python app.py
```

### Environment variables

| Variable | Required | Notes |
|---|---|---|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `FRONTEND_ORIGIN` | Yes | Allowed CORS origin, e.g. `https://rex-vms.vercel.app` |
| `SECURITY_PASSCODE` | First run | Seeds the Security portal passcode |
| `FRONTDESK_PASSCODE` | First run | Seeds the Front Desk portal passcode |
| `ADMIN_PASSCODE` | First run | Seeds the Admin portal passcode |
| `MISTRAL_API_KEY` | For AI help | Without it, `/api/ai/ask` returns a 503 |
| `MISTRAL_MODEL` | No | Defaults to `mistral-small-latest` |
| `PORT` | No | Defaults to `5000` |

The app refuses to start if `DATABASE_URL` or `FRONTEND_ORIGIN` is missing.

### Deploying on Render

Start command: `gunicorn app:app`

---

## Project status

Working MVP. Planned improvements: session expiry, login rate limiting, and visitor-list pagination.

---

## Author

Built by **Israel Oshifala** ([@Luthur24](https://github.com/Luthur24)).
