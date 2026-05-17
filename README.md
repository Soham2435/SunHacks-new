# 🛰️ Conflict Intelligence System (CPS)

> AI-powered OSINT conflict early-warning dashboard — built for SunHacks.

This system leverages a **Multi-Agent AI pipeline** to monitor open-source intelligence (OSINT) feeds, detect emerging conflicts, assess geopolitical risk in real-time, and generate scenario forecasts — all from a sleek, institutional-grade dashboard.

---

## ✨ Features

- 🔍 **Real-time OSINT Ingestion** — GNews API + BBC, Reuters, Al Jazeera RSS feeds
- 🤖 **4-Agent CrewAI Pipeline** — Ingestion → Detection → Simulation → Report
- 🗺️ **Interactive Risk Map** — Conflict hotspot visualization with Leaflet.js
- 📊 **Trend Analysis** — Historical risk/confidence charting per query
- 🌐 **Multi-Language Support** — English, Spanish, Hindi (i18next)
- 🔐 **Auth System** — Google OAuth + Email/Password login with bcrypt hashing + JWT sessions
- ☁️ **Dual Storage** — Local FAISS + SQLite · MongoDB Atlas cloud mirror
- 📱 **Fully Responsive** — Mobile-first design with glassmorphism UI

---

## Architecture

```
React (Vite)  ──→  FastAPI Backend  ──→  FAISS + SQLite (local)
                                    ──→  MongoDB Atlas (cloud mirror)
                                    ──→  CrewAI 4-Agent Pipeline
                                              └─→  Ollama / llama3.2
                                    ←──  JSON Intelligence Report
```

### Database Layer

| Store | Purpose | Technology |
|-------|---------|-----------|
| FAISS | Vector similarity search | `faiss-cpu` |
| SQLite | Article metadata + deduplication + trends | Python built-in |
| MongoDB Atlas | Cloud mirror, user storage, analysis records | `pymongo` |
| StoreManager | Unified orchestration interface | Custom |

**Key capabilities:**
- ✅ **Dual deduplication** — URL hash + content hash (never embeds the same article twice)
- ✅ **Incremental updates** — new vectors merged into existing index, no full rebuild
- ✅ **Disk persistence** — FAISS index saved after every update, loaded on startup
- ✅ **WAL mode** — concurrent-read safe SQLite configuration
- ✅ **Atlas fallback** — cloud cache used when GNews is unavailable

---

## Prerequisites

```bash
# 1. Install Ollama → https://ollama.ai
ollama pull llama3.2          # ~4 GB download

# 2. Python 3.11+
# 3. Node.js 18+
```

---

## Setup & Running

### 1. Backend

```powershell
cd backend

# Install Python dependencies
pip install -r requirements.txt

# Configure environment (copy and fill in your values)
# See .env section below for all supported variables

# Start Ollama in a separate terminal
ollama serve

# Start FastAPI
uvicorn main:app --reload --port 8000
```

### 2. Frontend

```powershell
cd frontend
npm install
npm run dev       # Opens at http://localhost:5173
```

---

## Environment Variables (`.env`)

Create a `backend/.env` file with the following keys:

```env
# News Ingestion
GNEWS_API_KEY=your_key_here          # Free at https://gnews.io (100 req/day)

# LLM / Ollama
OLLAMA_BASE_URL=http://localhost:11434
LLM_MODEL=llama3.2

# Auth
JWT_SECRET_KEY=your_super_secret_key
GOOGLE_CLIENT_ID=your_google_client_id.apps.googleusercontent.com

# MongoDB Atlas (optional — enables cloud mirroring & user persistence)
MONGO_URI=mongodb+srv://user:pass@cluster.mongodb.net/cps

# Dev flags
ENABLE_BYPASS_LOGIN=false            # Set true only in development
```

---

## API Endpoints

### System
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check + store statistics |
| POST | `/clear` | Wipe all stored data (dev/reset) |

### Ingestion & Analysis
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/update?query=` | Ingest OSINT (GNews + RSS) into FAISS + SQLite |
| GET | `/analyze?query=&lang=` | Run full AI analysis pipeline |
| GET | `/trends?query=` | Fetch historical risk/confidence data for a query |

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/login/google` | Verify Google ID token → return JWT |
| POST | `/auth/login/credentials` | Email + password login → return JWT |
| POST | `/auth/register` | Register new user with bcrypt-hashed password |
| GET | `/auth/me` | Return current user profile from JWT |
| POST | `/auth/login/bypass` | Dev-only bypass (requires `ENABLE_BYPASS_LOGIN=true`) |

---

## Example Usage

### Step 1 — Ingest news
```bash
curl -X POST "http://localhost:8000/update?query=Sudan+conflict"
```

### Step 2 — Analyze
```bash
curl "http://localhost:8000/analyze?query=Sudan+conflict&lang=en"
```

### Expected output
```json
{
  "risk": "HIGH",
  "risk_numerical": 82,
  "confidence": 0.82,
  "signals": ["military movement", "civilian displacement", "border clashes"],
  "location": "Khartoum, Sudan",
  "latitude": 15.5007,
  "longitude": 32.5599,
  "scenarios": {
    "best": "A UN-brokered ceasefire is negotiated within 30 days...",
    "worst": "Fighting spreads to Darfur; 2M additional refugees displaced...",
    "likely": "Continued SAF-RSF clashes with periodic ceasefires failing..."
  },
  "sources": ["https://bbc.com/...", "https://aljazeera.com/..."]
}
```

---

## Project Structure

```
SunHacks/
├── backend/
│   ├── main.py                     # FastAPI app — all routes & auth logic
│   ├── auth_utils.py               # JWT creation, Google token verification
│   ├── requirements.txt
│   ├── .env                        # Environment config (never commit secrets!)
│   ├── ingestion/
│   │   ├── gnews_fetcher.py        # GNews REST API (multi-language support)
│   │   └── rss_fetcher.py          # BBC, Reuters, Al Jazeera RSS feeds
│   ├── processing/
│   │   ├── cleaner.py              # HTML stripping + deduplication
│   │   └── chunker.py              # RecursiveCharacterTextSplitter
│   ├── database/
│   │   ├── sqlite_store.py         # Article metadata + trends (WAL mode)
│   │   ├── faiss_store.py          # FAISS with persistence + incremental merge
│   │   ├── mongo_store.py          # MongoDB Atlas cloud mirror + user store
│   │   └── store_manager.py        # Unified orchestration interface
│   ├── agents/
│   │   ├── ingestion_agent.py      # CrewAI: fact extractor
│   │   ├── detection_agent.py      # CrewAI: risk analyst
│   │   ├── simulation_agent.py     # CrewAI: scenario planner
│   │   └── report_agent.py         # CrewAI: JSON reporter
│   ├── pipeline/
│   │   └── crew_runner.py          # Single-shot CrewAI analysis pipeline
│   └── vector_store/               # Auto-created on first /update
│       ├── faiss_index/            # Persisted FAISS vectors
│       └── metadata.db             # SQLite article metadata + trends
│
├── frontend/
│   ├── index.html
│   ├── vite.config.js
│   ├── package.json
│   └── src/
│       ├── App.jsx                 # Root — routing + auth context
│       ├── Dashboard.jsx           # Main intelligence dashboard
│       ├── LoginPage.jsx           # Google OAuth + credentials login
│       ├── index.css               # Design system (glassmorphism)
│       ├── i18n.js                 # i18next multi-language config
│       ├── locales/                # en / es / hi translation files
│       ├── context/                # Auth context provider
│       └── components/
│           ├── Header.jsx
│           ├── ControlPanel.jsx
│           ├── RiskCard.jsx
│           ├── SignalsCard.jsx
│           ├── ScenariosCard.jsx
│           ├── SourcesCard.jsx
│           ├── TrendsChart.jsx
│           └── ConflictMap.jsx
│
├── render.yaml                     # Render.com deployment config
├── SETUP_GUIDE.md                  # Detailed local setup walkthrough
└── OLLAMA_TUNNEL_GUIDE.md          # Guide to tunnel Ollama for cloud deployment
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `503 Vector store empty` | Call `POST /update` first |
| `404 No documents found` | Try a broader keyword or check internet connection |
| Ollama connection error | Run `ollama serve` in a separate terminal |
| Slow response (~30–90s) | Normal — LLM processes 4 agents sequentially |
| GNews returning 0 docs | Check `GNEWS_API_KEY` in `.env` or quota exceeded |
| Google login fails | Verify `GOOGLE_CLIENT_ID` matches your OAuth app |
| MongoDB not connecting | Check `MONGO_URI` — system works without it (local-only mode) |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, Vite, Leaflet.js, i18next, Recharts |
| Backend | FastAPI, Python 3.11, uvicorn |
| AI Agents | CrewAI, Ollama, llama3.2 |
| Embeddings | HuggingFace `all-MiniLM-L6-v2` |
| Vector DB | FAISS (local) |
| Metadata DB | SQLite + MongoDB Atlas |
| Auth | Google OAuth 2.0, JWT (python-jose), bcrypt |
| Deployment | Render.com (backend), Cloudflare Pages (frontend) |

---

## License

MIT — built for SunHacks 2026.
