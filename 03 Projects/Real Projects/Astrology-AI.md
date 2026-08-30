# Astrology-AI

*Part of [[Projects]] → [[Real Projects]]*

**Repo:** [Astrology-AI](https://github.com/prakharDvedi/Astrology-AI)
**Visibility:** Public
**Language:** Python
**Description:** Vedic horoscope engine combining astronomical calculations, Neo4j knowledge graphs, and AI-generated interpretations.

---

A Neuro-Symbolic Vedic Astrology application that combines ancient astrological wisdom with modern AI.


## UI Screenshots
<img width="1574" height="833" alt="image" src="https://github.com/user-attachments/assets/6acc4394-80a7-4978-8fdd-fbbbc50fcc65" />
<img width="1728" height="868" alt="image" src="https://github.com/user-attachments/assets/154f5a1a-7b46-4d38-b4e1-9cbcbbafffcc" />



## Design System

The frontend uses a tokenized system for spacing, typography, color, radius, and motion. All interactive UI uses a single radius token, an 8-point spacing scale for layout, and a single font family with a defined type ramp.

## What It Is

PanditAI generates personalized Vedic horoscope readings by:

1. Calculating precise planetary positions using Swiss Ephemeris
2. Applying classical Vedic astrology rules from a knowledge graph
3. Synthesizing insights using AI (Groq/Ollama)

## Tech Stack

### Backend

- **FastAPI** - REST API server
- **Python** - Core logic
- **Swiss Ephemeris** - Astronomical calculations
- **Neo4j** - Knowledge graph for astrological rules
- **Groq API** - AI inference (with Ollama fallback)

### Frontend

- **Next.js** - React framework
- **Tailwind CSS** - Styling
- **TypeScript** - Type safety

## Features

- Precise astronomical calculations (Planets, Houses, Ascendant)
- Advanced techniques (Navamsa D9, Jaimini Karakas, Arudha Padas)
- AI-powered natural language readings
- Modern, responsive UI with dark theme
- Multiple Ayanamsa systems (Lahiri, Raman, KP)

## How It Works

```
User Input (Birth Details)
    ↓
Backend API (/calculate)
    ↓
├─ Astronomy Engine → Planetary Positions
├─ Knowledge Graph → Astrological Rules
└─ AI Model → Natural Language Reading
    ↓
JSON Response → Frontend Display
```

## Folder Structure

```
PanditAI/
├── backend/              # Python FastAPI server
│   ├── src/
│   │   ├── api/         # API endpoints
│   │   ├── astronomy/   # Calculation engine
│   │   ├── knowledge_graph/  # Neo4j queries
│   │   └── model/       # AI inference
│   ├── data/            # Ephemeris files
│   └── requirements.txt
│
├── frontend/            # Next.js application
│   ├── src/
│   │   ├── app/        # Pages & routes
│   │   └── components/ # UI components
│   └── package.json
│
└── docker-compose.yml   # Neo4j database
```

## Quick Start

### 1. Backend

```bash
cd backend
pip install -r requirements.txt
python setup_data.py  # Download ephemeris files
python -m uvicorn src.api.main:app --reload --port 8000
```

### 2. Frontend

```bash
cd frontend
npm install
npm run dev
```

Visit `http://localhost:3000`

### 3. Running with Docker (Recommended)

This sets up the Frontend, Backend, and Neo4j Database automatically.

1. Ensure **Docker Desktop** is installed and running.
2. Run the build command:

```bash
docker-compose up --build
```

3. The app will be available at `http://localhost:3000`.

## Environment Variables

Create `backend/.env`:

```
GROQ_API_KEY=your_groq_api_key_here
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password
```

## Testing & Validation Results (Benchmarked)

### 1. API Performance Test (FastAPI backend)

**What was tested:** Response latency of `/calculate` endpoint (Astrological computation + AI)
**How:** Sent sequential requests to the local production-like backend.
**Results:**

- **Initialization (Cold Start):** ~5.05s (Model loading & DB connection)
- **Warm Request (Median):** ~2.2s
- **Throughput:** Handling ~0.45 requests/sec (CPU-bound on local dev environment)

### 2. Cold Start & Deployment Behavior

**What was tested:** System behavior during initial boot vs steady state.
**How:** Measured first-request latency after server restart vs subsequent traffic.
**Results:**

- **Cold Start Latency:** 5456ms
- **Warm Latency:** 2220ms
- **Observation:** ~60% improved performance after cache warming. Validated `lru_cache` effectiveness for astrological calculations.

### 3. Docker Image Optimization Test

**What was tested:** `backend` image build configuration.
**How:** Audited `requirements.txt` for unnecessary GPU dependencies.
**Results:**

- **Optimization:** Enforced `torch --extra-index-url https://download.pytorch.org/whl/cpu`.
- **Impact:** Reduced build size by ~700MB (excluding CUDA binaries), significantly speeding up Render deployment and reducing monthly storage costs.

### 4. Frontend Performance Test (Next.js)

**What was tested:** UI Interactivity and Load time.
**Action Required:** Run Lighthouse in Chrome DevTools on `localhost:3000`.
**Expecting:**

- **FCP:** < 1.5s (Static Site Generation/SSR)
- **SEO:** 100/100 (Semantic HTML5)

### 5. External Dependency Failure Test

**What was tested:** Resilience to Database/AI outages.
**How:** (Planned) Temporarily disable Network Adapter or Stop Neo4j Container.
**Logic:** Application should return `503 Service Unavailable` or `500` with clear JSON error message, not hang indefinitely.

### 6. Contract Validation

**What was tested:** API Schema consistency.
**How:** Validated `schemas.py` Pydantic models against Frontend `types.ts`.
**Results:** Strong typing ensures `BirthDetails` payload structure is strictly enforced, preventing runtime type errors.
