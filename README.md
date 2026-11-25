# PERIODIX — Intelligent Periodic Table & Chemistry Learning Platform

PERIODIX is a full-stack web application combining a **React + Vite + Tailwind CSS + Framer Motion + Recharts** scientific interface with a **FastAPI + SQLAlchemy + SQLite + Scikit-learn** backend.

---

## Implemented Modules (All 7 Phases Complete)

1. **Telemetry Dashboard (`/`)**: Live SQLite learner metrics (quizzes completed, accuracy %, XP, streak, strongest/weakest topics, recent activity, Recharts accuracy curve, and editable learner profile).
2. **118-Element Interactive Periodic Table (`/periodic-table`)**: Complete schema-validated IUPAC/NIST dataset (`1 H` – `118 Og`), search by name/symbol/atomic number, color-coded family chips, group/period/physical-state filters, Grid vs Compact Card views, and detailed atomic inspection drawer.
3. **Quiz Arena (`/quiz`)**: Multi-difficulty (`easy`, `medium`, `advanced`) timed multiple-choice chemistry sessions, instant scientific feedback & explanations, XP awards, and SQLite attempt history.
4. **Adaptive Quiz Engine (`/quiz`)**: Scales difficulty up after 2 consecutive correct answers, reduces difficulty after 2 consecutive errors, tracks topic mastery, and prevents repeating recently answered questions.
5. **Random Forest ML Recommendation Engine (`/recommendations`)**: `RandomForestClassifier` analyzing response times, topic accuracy, difficulty, and streaks; cold-start fallback below 10 real question responses; strict separation between held-out synthetic benchmark validation and real learner SQLite data.
6. **Periodic Trends Laboratory (`/trends`)**: Interactive Recharts visualizations for atomic radius, electronegativity, first ionization energy, and atomic mass, plus side-by-side multi-element comparison and quantum explanations.
7. **Achievements & Profile (`/achievements`, `/profile`)**: XP rank tiers, learning streaks, unlockable milestone badges, full quiz history, and system verification telemetry.

---

## Setup & Local Hosting

### 1. Backend (FastAPI + SQLite + Scikit-learn)

From `periodix/backend`:

```powershell
# Create & activate virtual environment (Python 3.14)
py -m venv .venv
.\.venv\Scripts\Activate.ps1

# Install dependencies
pip install -r requirements.txt

# Run automated test suite (13 tests across all modules)
pytest -v

# Start FastAPI backend on http://127.0.0.1:8000
uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Interactive OpenAPI docs: `http://127.0.0.1:8000/docs`

### 2. Frontend (React + Vite + Tailwind CSS)

From `periodix/frontend`:

```powershell
# Install npm packages
npm install

# Run development server (proxies /api/* to http://127.0.0.1:8000)
npm run dev -- --host 127.0.0.1 --port 5173

# Build production static assets
npm run build
```

---

## Core API Endpoints

- `GET /api/health` — Backend & SQLite connectivity check
- `GET /api/dashboard/summary` & `PATCH /api/dashboard/profile` — Telemetry & profile updates
- `GET /api/elements`, `GET /api/elements/{atomic_number}`, `GET /api/elements/symbol/{symbol}` — 118 validated elements
- `GET /api/quiz/meta`, `POST /api/quiz/start`, `POST /api/quiz/answer`, `GET /api/quiz/history` — Adaptive quiz engine
- `GET /api/recommendations` — Cold-start fallback or live `RandomForestClassifier` predictions + held-out evaluation
- `GET /api/trends`, `GET /api/trends/compare?ids=...` — Periodic trends & element comparison
- `GET /api/achievements` — XP rank & badge ledger
