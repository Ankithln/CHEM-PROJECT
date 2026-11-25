# PERIODIX — Public Deployment Guide (Vercel + Render)

This guide covers deploying **PERIODIX** for public production use:
- **Frontend:** React + Vite SPA hosted on **Vercel**
- **Backend:** Python + FastAPI + SQLAlchemy + Scikit-learn API hosted on **Render**

> **Security Note:** Never commit real `.env` files, database passwords, or connection strings to Git. Configure all sensitive values directly inside the Render and Vercel dashboards.

---

## 1. Production Environment Variables

### Backend (`periodix-api` on Render)

| Variable | Required | Example / Format | Purpose |
| --- | --- | --- | --- |
| `DATABASE_URL` | Yes (Production) | `postgresql://USER:PASS@HOST:5432/DB` or `sqlite:////var/data/periodix.db` | Persistent database connection string (`postgres://` and `postgresql://` automatically normalize to `postgresql+psycopg://`) |
| `CORS_ORIGINS` | Yes | `https://your-periodix.vercel.app` | Comma-separated (or JSON array) allowed frontend origins |
| `CORS_ORIGIN_REGEX` | Optional | `https://.*\.vercel\.app` | Allows Vercel preview deployment subdomains if desired |
| `MIN_ATTEMPTS_FOR_ML` | Optional | `10` | Minimum real answered quiz questions before Random Forest personalization activates |

Template: [backend/.env.example](backend/.env.example)

### Frontend (`periodix` on Vercel)

| Variable | Required | Example / Format | Purpose |
| --- | --- | --- | --- |
| `VITE_API_BASE_URL` | Yes | `https://periodix-api.onrender.com` | Public HTTPS URL of your Render backend (no trailing slash) |

Template: [frontend/.env.example](frontend/.env.example)

---

## 2. Production Database Persistence Recommendation

By default during local development, PERIODIX uses `sqlite:///./periodix.db` inside `backend/`. On Render's standard container filesystem, local files are **ephemeral** and reset whenever the service restarts or redeploys.

Choose one of these production-safe options:

### Option A: Managed PostgreSQL (Recommended)
1. Create a **PostgreSQL** database on Render (or Neon / Supabase).
2. Copy the **Internal Database URL** (if hosted on Render) or **External Connection String**.
3. Set it as `DATABASE_URL` in your Render Web Service environment variables.
4. `backend/app/config.py` automatically maps `postgres://` / `postgresql://` to `postgresql+psycopg://`, `backend/app/database.py` enables `pool_pre_ping=True`, and tables + default learner profile initialize automatically on startup via FastAPI's lifespan hook.

### Option B: Persistent SQLite on a Mounted Render Disk
1. Attach a **Persistent Disk** to your Render Web Service mounted at `/var/data` (requires a paid Render instance tier).
2. Set `DATABASE_URL=sqlite:////var/data/periodix.db`.
3. `backend/app/database.py` automatically creates parent directories if needed.

---

## 3. Deploy Backend to Render

You can deploy either via the included [render.yaml](render.yaml) Blueprint or manually:

### Manual Setup on Render
1. Push your `periodix` repository to GitHub (`.gitignore` excludes `.venv`, `node_modules`, `*.db`, and `.env` files).
2. In the Render Dashboard, click **New + -> Web Service** and connect your GitHub repository.
3. Configure the service settings:
   - **Root Directory:** `backend`
   - **Runtime:** `Python 3`
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
   - **Health Check Path:** `/api/health`
4. Add environment variables:
   - `PYTHON_VERSION`: `3.12.8`
   - `DATABASE_URL`: `<your managed PostgreSQL connection string>`
   - `CORS_ORIGINS`: `http://localhost:5173` (you will update this with your Vercel URL right after Step 4)
5. Deploy and verify `https://<your-render-service>.onrender.com/api/health` returns:
   ```json
   {
     "status": "online",
     "database": "connected",
     "current_phase": 7
   }
   ```

---

## 4. Deploy Frontend to Vercel

1. In the Vercel Dashboard, click **Add New... -> Project** and select the same GitHub repository.
2. Configure Project Settings:
   - **Framework Preset:** `Vite`
   - **Root Directory:** `frontend`
   - **Build Command:** `npm run build`
   - **Output Directory:** `dist`
3. Add Environment Variable:
   - `VITE_API_BASE_URL` = `https://<your-render-service>.onrender.com` *(no trailing slash)*
4. Click **Deploy**.
   - Client-side React Router routes (`/periodic-table`, `/explore-periodic-table`, `/quiz`, `/trends`, `/recommendations`, `/achievements`, `/profile`) are automatically handled by [frontend/vercel.json](frontend/vercel.json).

---

## 5. Finalize CORS Between Vercel and Render

1. Copy your deployed Vercel production domain (for example, `https://periodix-chemistry.vercel.app`).
2. Open your Render Web Service -> **Environment**:
   - Set `CORS_ORIGINS` to `https://periodix-chemistry.vercel.app` (or comma-separated if using a custom domain too).
   - Optionally set `CORS_ORIGIN_REGEX` to `https://.*\.vercel\.app` if you want Vercel preview URLs to work as well.
3. Trigger a manual deploy / restart on Render so the updated CORS settings take effect.

---

## 6. Post-Deployment Verification Checklist

1. Open `https://<your-vercel-domain>/` and confirm the Dashboard loads live telemetry from Render (`Status: Connected`).
2. Refresh `https://<your-vercel-domain>/periodic-table` and `/explore-periodic-table` directly in the browser to verify `vercel.json` SPA rewrites work without 404 errors.
3. Inspect an element on the Periodic Table and plot a trend in the Trends Laboratory (`/trends`).
4. Complete a 5-question quiz in the Quiz Arena (`/quiz`), refresh the page, and confirm the attempt persists in Quiz History and Profile (`/profile`).
5. Visit AI Recommendations (`/recommendations`) to verify cold-start fallback (before 10 responses) and Random Forest predictions (after 10+ responses).
