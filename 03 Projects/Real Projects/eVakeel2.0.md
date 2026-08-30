# eVakeel2.0

*Part of [[Projects]] → [[Real Projects]]*

**Repo:** [eVakeel2.0](https://github.com/prakharDvedi/eVakeel2.0)
**Visibility:** Public
**Language:** JavaScript
**Description:** Legal document QA platform using semantic search, RAG pipelines, and AI-assisted case analysis.

---

# eVakeel

eVakeel is an AI-assisted legal information platform focused on Indian law. The current codebase is built around a React frontend, a Fastify backend, a Python embedding service for retrieval, and a local legal corpus indexed into vectors.

This README reflects the code that is actually present in the repository.

## What The Project Does

The live code currently supports three main user-facing flows:

- Legal advisor chat with retrieval-augmented context from an Indian law corpus
- Legal document upload and analysis for PDFs and image uploads
- A legal score checklist UI that links users to official public portals

The backend also contains extra routes for templates, incidents, sessions, and admin actions, but several of those are partial or depend on integrations that are currently disabled.

## Actual Tech Stack

### Frontend

- React 19
- Vite 7
- React Router 7
- Tailwind CSS 4
- Framer Motion
- React Textarea Autosize

Frontend code lives in `frontend/`.

### Main Backend

- Node.js
- Fastify
- `@fastify/cors`
- `@fastify/multipart`
- `@fastify/websocket`
- Gemini via `@google/generative-ai`
- `pdf-parse`
- Redis client

Backend code lives in `Backend/`.

### Retrieval / Embedding Service

- Python
- Flask
- Hugging Face Transformers
- Torch
- `law-ai/InLegalBERT` for embeddings

The embedding service is implemented in `Backend/rag_server.py`.

### Data Layer

- Local legal corpus in `Backend/data/raw/*.txt`
- Local vector store in `Backend/data/vectors.json`
- Redis for caching retrieval results

## High-Level Architecture

The project is not a single backend. It is effectively a multi-service stack:

1. `frontend/` renders the UI and calls the backend over HTTP.
2. `Backend/server.js` exposes REST and WebSocket endpoints.
3. `Backend/services/ragService.js` performs retrieval against the local vector store.
4. `Backend/rag_server.py` generates embeddings for retrieval using InLegalBERT.
5. `Backend/services/aiProxy.js` sends prompts to Gemini for chat and document analysis.
6. `Backend/services/cacheService.js` caches retrieval results in Redis.

## Request Flow

### Legal Chat

1. The user sends a message from the Legal Advisor page.
2. The frontend calls `POST /api/chat`.
3. The backend extracts the latest user query.
4. Redis is checked for cached retrieval results.
5. On a cache miss, the backend calls the Python embedding server.
6. The query embedding is compared against `Backend/data/vectors.json`.
7. Top legal chunks are added to the Gemini prompt.
8. Gemini returns the final answer.

### Document Analysis

1. The user uploads a file through the Document Parser page.
2. The frontend calls `POST /documents/upload`.
3. The backend stores the file locally unless Firebase storage is wired in.
4. The frontend then calls `POST /documents/analyze`.
5. The backend extracts text from the PDF, retrieves related legal context, and asks Gemini to produce an analysis.

## Repository Structure

```text
eVakeel_MUJhackX/
├── Backend/
│   ├── data/
│   │   ├── raw/                  # Source legal text corpus
│   │   └── vectors.json          # Generated vector store
│   ├── plugins/                  # Fastify plugins and integration scaffolding
│   ├── routes/                   # API routes
│   ├── scripts/                  # Corpus ingestion and verification scripts
│   ├── services/                 # AI, RAG, cache, and processing logic
│   ├── rag_server.py             # Python embedding service
│   ├── server.js                 # Fastify entry point
│   └── requirements.txt          # Python dependencies
├── frontend/
│   ├── public/                   # Static assets
│   ├── src/
│   │   ├── components/           # Shared React components
│   │   ├── pages/                # Route-level pages
│   │   ├── services/             # Frontend API client
│   │   └── utils/                # Frontend utilities
│   ├── package.json
│   └── vite.config.js
├── docker-compose.yml            # Present, but not fully aligned with repo state
└── setup_vm.sh                   # VM bootstrap script
```

## Important Backend Routes

### Working Core Routes

- `POST /api/chat` - main legal advisor chat endpoint
- `POST /documents/upload` - upload a document
- `POST /documents/analyze` - analyze a document by file path
- `GET /health` - backend health check
- `GET /ws/chat` - WebSocket chat endpoint

### Extra / Partial Routes

- `/retrieve`
- `/templates`
- `/incidents`
- `/sessions`
- `/admin`

Some of these exist as scaffolding and are not fully wired end to end.

## Environment Variables

### Backend

Create `Backend/.env`:

```env
GROQ_API_KEY=your_groq_api_key
GROQ_TEXT_MODEL=llama-3.3-70b-versatile
GROQ_VISION_MODEL=meta-llama/llama-4-scout-17b-16e-instruct
PORT=5050
PYTHON_RAG_URL=http://127.0.0.1:8000/embed
REDIS_URL=redis://localhost:6379
```

Notes:

- `GROQ_API_KEY` is required for chat and analysis.
- `PYTHON_RAG_URL` defaults to `http://127.0.0.1:8000/embed` if omitted.
- `REDIS_URL` is optional in intent, but the current cache client will attempt to connect on startup.

### Frontend

Create `frontend/.env`:

```env
VITE_API_URL=http://localhost:5050
```

## Local Development Setup

### Prerequisites

- Node.js 18+ recommended
- Python 3.10+
- Redis
- A valid Gemini API key

### 1. Install Frontend Dependencies

```bash
cd frontend
npm install
```

### 2. Install Backend Dependencies

```bash
cd Backend
npm install
```

### 3. Install Python Dependencies

```bash
cd Backend
pip install -r requirements.txt
```

### 4. Start Redis

Run Redis locally on `6379`.

### 5. Start The Python Embedding Service

```bash
cd Backend
python rag_server.py
```

This starts the embedding server on `http://localhost:8000`.

### 6. Build Or Refresh The Vector Store

If you change the text files inside `Backend/data/raw`, rebuild the vector store:

```bash
cd Backend
node scripts/ingestCorpus.js
```

This script reads the raw legal text files, chunks them, requests embeddings from the Python service, and writes `Backend/data/vectors.json`.

### 7. Start The Fastify Backend

```bash
cd Backend
npm start
```

The backend defaults to port `5050`.

### 8. Start The Frontend

```bash
cd frontend
npm run dev
```

## Scripts

### Backend

- `npm start` - starts the Fastify server
- `npm test` - runs `test_integration.js`

### Frontend

- `npm run dev` - starts the Vite dev server
- `npm run build` - builds the frontend
- `npm run lint` - runs ESLint
- `npm run preview` - previews the production build

## Current Limitations

These are important if you plan to extend or deploy the project:

- Firebase-related persistence exists in code but is mostly disabled or not registered in the active server boot path.
- Session routes depend on Firestore, but Firestore is not currently initialized in `Backend/server.js`.
- Some routes such as templates, incidents, and admin actions appear to be incomplete or rely on unsupported `aiProxy` paths.
- `docker-compose.yml` does not fully match the current repository layout.
- Port references are inconsistent in some deployment files (`5000` vs `5050`).
- Document analysis is implemented primarily around PDF text extraction; image handling is present at the API shape level but not fully realized in the analysis path.

## Files That Matter Most

If you are trying to understand or modify the project, start here:

- `frontend/src/App.jsx`
- `frontend/src/services/api.js`
- `frontend/src/pages/LegalAdvisorPage.jsx`
- `frontend/src/pages/DocumentParserPage.jsx`
- `Backend/server.js`
- `Backend/routes/chat.js`
- `Backend/routes/documents.js`
- `Backend/services/aiProxy.js`
- `Backend/services/ragService.js`
- `Backend/services/cacheService.js`
- `Backend/rag_server.py`
- `Backend/scripts/ingestCorpus.js`

## Status Summary

The strongest, most coherent part of the project is the legal chat and document-analysis pipeline:

- React frontend
- Fastify API
- Gemini prompting
- InLegalBERT embeddings via Python
- local legal corpus retrieval
- Redis caching

The persistence and auxiliary platform features are present mostly as unfinished scaffolding.

## Disclaimer

This project is an informational tool. It does not replace advice from a qualified lawyer.

**Related:** [[eVakeel_MUJhackX]] — earlier hackathon iteration of this project.
