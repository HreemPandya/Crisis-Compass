# CrisisCompass

Location-aware **emergency monitoring** from public news-style feeds. The browser supplies coordinates; the **Flask** backend reverse-geocodes, scrapes RSS and related sources, and ranks items with **keyword + spaCy** heuristics and trust signals. Heavy work stays on the server.

---

## Features

- **Incidents** — Ranked cards with severity (icons + text), source links, filters and sort (points, severity, trust, time).
- **Local feeds** — `POST /get-local-incidents` with lat/lng; deduplication by URL or title+location; results merged into SQLite so an empty scrape does not wipe prior rows.
- **Geolocation** — Longer timeouts, optional **Permissions API** retry when access is granted after load, and **Try location again** without a full page reload.
- **Reports** — Time windows, comparison periods, Recharts (severity over time, type, sources, trust), CSV export, printable HTML, optional **LangChain + OpenAI** summary and entity graph (requires `OPENAI_API_KEY`).
- **Settings** — Severity floor, quiet-hours notice, AI report options, accessibility (font scale, reduced motion); stored in **localStorage**.
- **Dev workflow** — `npm run dev` runs Flask and Vite together; the UI calls `/api/...` via the Vite proxy.

---

## Tech stack

| Layer    | Stack |
|----------|--------|
| Backend  | Python 3.9+, Flask, Flask-CORS, BeautifulSoup, spaCy, feedparser, geopy, requests, SQLite, python-dotenv, LangChain + OpenAI (optional) |
| Frontend | React 18, Vite, PostCSS/Tailwind tooling, Lucide React, Axios, Recharts |

---

## Setup

### Prerequisites

- Python 3.9+
- Node.js 18+ recommended

### Steps

1. Clone the repo and open the project root.

2. **Backend dependencies** (from project root — `setup_backend.py` changes into `backend/`):

   ```bash
   python setup_backend.py
   ```

3. **Frontend dependencies**:

   ```bash
   npm install
   ```

4. **Optional — AI reports**  
   Copy [`.env.example`](.env.example) to **`.env`** in the project root and set `OPENAI_API_KEY`. The API loads `.env` automatically (see [`backend/app.py`](backend/app.py)). Restart the backend after changes.

5. **Run** (Flask + Vite):

   ```bash
   npm run dev
   ```

6. Open **http://localhost:5173** (or the URL Vite prints).

### npm scripts

| Script        | Command |
|---------------|---------|
| `npm run dev` | Flask + Vite together |
| `npm run dev:api` | Flask only (`python backend/app.py`) |
| `npm run dev:web` | Vite only |
| `npm run build` | Production frontend build into `dist/` |
| `npm run lint`  | ESLint |

On Windows you can also use [`start.bat`](start.bat) to launch backend and frontend in separate windows.

### Manual backend (without `setup_backend.py`)

```bash
cd backend
pip install -r requirements.txt
python -m spacy download en_core_web_sm
python app.py
```

---

## **How It Works**


---

## How it works

1. The app requests geolocation and sends coordinates to **`POST /get-local-incidents`** (or falls back to the general list).
2. The server reverse-geocodes, builds regional RSS queries, parses entries, scores text, and merges rows into **SQLite** (plus an in-memory cache for API responses).
3. The React app talks to **`/api/...`** (Vite proxies to Flask in dev). For production, build with the right `VITE_API_URL` and serve `dist/` with the API (see [DEPLOY.md](DEPLOY.md) if present in your tree).

---

## API (Flask)

- `GET /health` — Liveness
- `GET /get-incidents` — All stored incidents
- `POST /get-local-incidents` — Body: `{ "latitude", "longitude" }` — scrape + merge; response is the **full** incident store after merge
- `POST /scrape` — Manual URL scrape (legacy)
- `GET /debug/logs` — Debug / last scrape hint
- `GET /report/summary?hours=24&compare_hours=` — Deterministic aggregates
- `GET /report/export.csv?hours=24` — CSV export
- `GET /report/print.html?hours=24` — Printable HTML
- `POST /report/insights` — JSON body: `hours`, optional `compare_hours`, `include_llm`, `tone`, `length`


- Coordinates are sent to **your** backend for geocoding and feed selection. Incident rows live in **local SQLite** on the server; plan retention and backups for production.
- Respect robots.txt, terms of use, and rate limits for news sources you query.
- Do not commit **`.env`**; it is gitignored. Use [`.env.example`](.env.example) as a template.
