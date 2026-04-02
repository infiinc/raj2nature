# 🌿 Raj2 Nature — Full-Stack Deployment Guide

**Stack:** React (Vite + Tailwind) · FastAPI (Python) · Neon (PostgreSQL)

---

## 📁 Project Structure

```
raj2nature/
├── frontend/          # React + Vite + Tailwind CSS
│   ├── src/
│   │   ├── App.jsx    # Main app (all pages)
│   │   └── api.js     # API client (all fetch calls)
│   ├── .env.example
│   └── package.json
│
└── backend/           # FastAPI + psycopg2
    ├── main.py        # All routes (auth + bookings)
    ├── database.py    # Neon connection + schema init
    ├── requirements.txt
    └── .env.example
```

---

## 1️⃣  Set Up Neon (Database)

1. Go to **https://console.neon.tech** → Sign up / Log in
2. Click **"New Project"** → give it a name (e.g. `raj2nature`)
3. Once created, click **"Connection Details"**
4. Copy the **Connection string** — it looks like:
   ```
   postgresql://alex:AbC123dEf@ep-cool-darkness-123456.us-east-2.aws.neon.tech/neondb?sslmode=require
   ```
5. Keep this string handy; you'll paste it as `DATABASE_URL` below.

> The tables (`users`, `bookings`) are created automatically when the backend first starts.

---

## 2️⃣  Run the Backend Locally

```bash
cd backend

# Copy and edit the env file
cp .env.example .env
# → Paste your Neon DATABASE_URL, set a random JWT_SECRET

# Install dependencies
pip install -r requirements.txt

# Start the server (runs on http://localhost:8000)
uvicorn main:app --reload
```

**Env variables (`backend/.env`):**

| Variable       | Description                                          |
|----------------|------------------------------------------------------|
| `DATABASE_URL` | Neon PostgreSQL connection string                    |
| `ADMIN_EMAIL`  | Email that gets admin role on signup (default: `admin@raj2nature.com`) |
| `JWT_SECRET`   | Long random string — run `python -c "import secrets; print(secrets.token_hex(32))"` |
| `FRONTEND_URL` | Your frontend URL for CORS (e.g. `http://localhost:5173`) |

---

## 3️⃣  Run the Frontend Locally

```bash
cd frontend

# Copy env file (leave VITE_API_URL empty to use the proxy)
cp .env.example .env

# Install dependencies
npm install

# Start dev server (runs on http://localhost:5173)
npm run dev
```

The Vite dev server proxies `/api` → `http://localhost:8000` automatically.

---

## 4️⃣  Deploy to Production

### Backend → Render (free tier)

1. Push your code to GitHub
2. Go to **https://render.com** → New → **Web Service**
3. Connect your GitHub repo, set **Root Directory** to `backend`
4. **Build command:** `pip install -r requirements.txt`
5. **Start command:** `uvicorn main:app --host 0.0.0.0 --port $PORT`
6. Add **Environment Variables**:
   - `DATABASE_URL` = your Neon connection string
   - `JWT_SECRET` = your random secret
   - `ADMIN_EMAIL` = admin@raj2nature.com
   - `FRONTEND_URL` = your Vercel frontend URL (add after deploy)

### Frontend → Vercel (free tier)

1. Go to **https://vercel.com** → New Project → import your repo
2. Set **Root Directory** to `frontend`
3. Add **Environment Variable**:
   - `VITE_API_URL` = your Render backend URL (e.g. `https://raj2nature-api.onrender.com`)
4. Deploy!

---

## 🔐 Default Accounts

| Email                    | Password   | Role   |
|--------------------------|------------|--------|
| `admin@raj2nature.com`   | (set yours)| Admin  |
| any other email          | (set yours)| Member |

The first person to **register** with the `ADMIN_EMAIL` address gets admin access.

---

## 🗄️ Database Schema

```sql
-- Created automatically on startup
CREATE TABLE users (
    id            SERIAL PRIMARY KEY,
    name          VARCHAR(255) NOT NULL,
    email         VARCHAR(255) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    role          VARCHAR(50) DEFAULT 'member',
    created_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE bookings (
    id         SERIAL PRIMARY KEY,
    user_id    INTEGER REFERENCES users(id) ON DELETE CASCADE,
    user_email VARCHAR(255) NOT NULL,
    date       DATE NOT NULL,
    time_slot  VARCHAR(100) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    details    TEXT DEFAULT '',
    status     VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 🔌 API Endpoints

| Method | Path                             | Auth     | Description              |
|--------|----------------------------------|----------|--------------------------|
| POST   | `/api/auth/register`             | None     | Create account           |
| POST   | `/api/auth/login`                | None     | Login, get JWT           |
| GET    | `/api/auth/me`                   | Member+  | Get current user         |
| POST   | `/api/bookings`                  | Member+  | Create booking request   |
| GET    | `/api/bookings/my`               | Member+  | My bookings              |
| GET    | `/api/bookings/all`              | Admin    | All bookings             |
| PATCH  | `/api/bookings/{id}/status`      | Admin    | Approve / reject         |
| GET    | `/api/health`                    | None     | Health check             |

Interactive API docs (Swagger): **http://localhost:8000/docs**
