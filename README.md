# African Solution Team App

Multi-user team workspace: shared kanban boards (like Trello), login,
and admin-controlled permissions. Built for the African Solution team —
the Somali plastic-recycling company turning plastic waste into building
materials — working from different locations (Mogadishu, Kismayo, and the US).

**Af-Soomaali:** App kooxeed — boards la wadaago (sida Trello), login,
admin-kuna wuxuu maamulaa cidda soo gasha. Kooxda African Solution
meel kasta ha joogtee way isticmaali kartaa marka la deploy-gareeyo.

## Features

- Sign up / Log in (first registered user automatically becomes **admin**)
- Kanban boards: create lists, cards, drag & drop between lists
- Card details: description, assignee, due date (overdue highlighted)
- Admin panel: add users directly, promote/demote, activate/deactivate,
  permanently delete users (cannot delete yourself or the last admin)
- Board members: admin invites members to boards, assigns board roles
- Bilingual UI: Somali / English toggle
- Sessions last 30 days; friendly "server waking up" screen on cold starts
- Refuses to start without `DATABASE_URL` (no silent data loss)

## Run locally

```bash
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
ALLOW_SQLITE=true SECRET_KEY=dev-secret \
  uvicorn backend.app:app --host 0.0.0.0 --port 8000 --app-dir .
# open http://localhost:8000
```

`ALLOW_SQLITE=true` is for local development only. In production the app
**fails fast** if `DATABASE_URL` is missing, so data can never silently
fall back to an ephemeral SQLite file.

## Deploy (Render + Neon PostgreSQL, free)

### Step 1 — Neon PostgreSQL database (free, xog joogto ah)

1. Go to **neon.tech** → log in → open your project (or create one).
2. Click **Connect** → choose **Connection string** (NOT the `psql` command)
   → **copy** the full string (starts with `postgresql://`).
3. Keep this string secret — it contains the database password.

### Step 2 — Web service (Render.com, free)

1. Push this folder to a GitHub repo (e.g. `african-solution-team-app`).
2. In Render: **New → Web Service** → connect the repo.
   - Build command: `pip install -r requirements.txt`
   - Start command: `uvicorn backend.app:app --host 0.0.0.0 --port $PORT --app-dir .`
3. **Environment** tab → add:
   - Key: `DATABASE_URL`, Value: the Neon connection string you copied
   - Key: `SECRET_KEY`, Value: a long random string (keep it secret —
     without it, everyone is logged out on every restart)
4. **Save** → Render redeploys automatically. Tables are created on first boot.
5. Open the URL, **sign up first** — that account becomes admin.
   Then invite the team (they sign up, you activate/promote them in Admin panel).

⚠️ Do **not** create a second Render service for this repo — one service,
one URL, one set of environment variables. Confusion starts with duplicates.

### Option B — Railway / Fly.io / VPS

Same code works anywhere Docker runs:

```bash
docker build -t african-solution-team-app .
docker run -p 8000:8000 -e SECRET_KEY=your-long-secret \
  -e DATABASE_URL=postgresql://user:pass@host/dbname \
  african-solution-team-app
```

## Security notes

- `SECRET_KEY` and `DATABASE_URL` come from environment variables only.
  Never commit real secrets.
- Passwords are hashed with PBKDF2-SHA256 (200k iterations).
- Deactivated users cannot log in; removing a board member revokes access.
- For production, serve behind HTTPS (Render/Railway do this automatically).
