# 🤖 AI Chatbot System

> A production-grade, RAG-powered AI chatbot with multi-agent architecture, eCommerce integrations, and enterprise security — built with FastAPI, LangGraph, and React.

[![CI](https://github.com/your-org/AI_Chatbot_System/actions/workflows/ci.yml/badge.svg)](https://github.com/your-org/AI_Chatbot_System/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green.svg)](https://fastapi.tiangolo.com/)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Delivery Phases](#delivery-phases)
- [Quick Start](#quick-start)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
- [Testing](#testing)
- [CI/CD](#cicd)
- [Contributing](#contributing)
- [Naming Conventions](#naming-conventions)

---

## Overview

The AI Chatbot System is a full-stack, modular chatbot platform designed for eCommerce and enterprise use cases. It combines Retrieval-Augmented Generation (RAG), a LangGraph multi-agent system, and a security-first architecture — all delivered across three phased sprints.

**Key capabilities:**

- RAG pipeline with ChromaDB/Qdrant vector stores and source citation
- LangGraph multi-agent system with intent routing and specialised sub-agents
- JWT auth, RBAC, prompt injection guards, PII scrubbing, and audit logging
- Shopify / WooCommerce / Stripe / Razorpay integrations
- Prometheus metrics, OpenTelemetry tracing, and Grafana dashboards
- Voice interface via Deepgram (STT) and ElevenLabs (TTS)
- React chat widget and admin panel
- 4-tier test suite: unit → integration → RAG eval → load/security

---

## Architecture

```
User → React ChatWidget (frontend/)
         ↓ HTTPS
FastAPI App (app.py)
  ├── Routers (routers/)         — REST endpoints per resource
  ├── Security (security/)       — Auth, RBAC, input validation, PII
  ├── RAG Pipeline (rag/)        — Ingest → Chunk → Embed → Retrieve
  ├── Agents (agents/)           — LangGraph StateGraph, supervisor routing
  ├── Tools (tools/)             — LangChain @tool functions
  ├── Models (models/)           — Pluggable LLM backends (OpenAI / Claude / Ollama)
  ├── Integrations (integrations/) — Shopify, Stripe, Twilio, Slack, etc.
  ├── Database (database/)       — SQLAlchemy ORM, Redis cache, Alembic migrations
  ├── Prompts (prompts/)         — System prompts & templates (admin-editable)
  └── Observability (observability/) — Logging, metrics, tracing, health checks
```

**Design principles:**

- **Separation of Concerns** — every directory owns exactly one responsibility
- **Swappable Backends** — swap any LLM, vector store, or third-party API without touching application logic
- **Security by Design** — 10 dedicated security files; auth, RBAC, and prompt guards are first-class
- **Observability First** — Prometheus, OpenTelemetry, and Grafana dashboards ship from MVP
- **Phase-Gated Delivery** — files tagged MVP / P2 / P3 so value ships every sprint

---

## Project Structure

```
AI_Chatbot_System/          — 70 files · 15 modules
├── app.py                  [MVP] FastAPI app factory & router registration
├── config.py               [MVP] Pydantic Settings; all env vars & feature flags
├── utils.py                [MVP] Shared helpers: chunking, sanitisation, hashing
├── requirements.txt        [MVP] All Python dependencies (pinned)
├── .env.example            [MVP] Environment variable template
├── docker-compose.yml      [MVP] API server, Redis, Qdrant, Postgres, Grafana
├── Dockerfile              [MVP] Multi-stage build → slim runtime image
│
├── models/                 — LLM providers (pluggable .generate() interface)
│   ├── openai_model.py     [MVP]
│   ├── anthropic_model.py  [MVP]
│   ├── hf_model.py         [MVP] Ollama/Llama 3 local fallback
│   ├── model_router.py     [P2]  Circuit-breaker fallback chain
│   └── fine_tuned_model.py [P3]  LoRA/QLoRA adapter loader
│
├── rag/                    — End-to-end RAG pipeline
│   ├── document_loader.py  [MVP] PDF, DOCX, TXT ingestion
│   ├── chunker.py          [MVP] Recursive & semantic chunking
│   ├── embedder.py         [MVP] OpenAI embeddings + HuggingFace fallback
│   ├── vector_store.py     [MVP] ChromaDB (dev) / Qdrant (prod)
│   ├── retriever.py        [MVP] Top-k similarity search
│   ├── reranker.py         [P2]  Cohere Rerank / cross-encoder
│   ├── citation_builder.py [MVP] Source file + page number citations
│   └── pipeline.py         [MVP] Full ingest → embed → retrieve → respond
│
├── agents/                 — LangGraph multi-agent system
│   ├── graph.py            [P2]  StateGraph nodes, edges, routing
│   ├── state.py            [P2]  Typed AgentState dataclass
│   ├── supervisor.py       [P2]  Intent classifier & router
│   ├── rag_agent.py        [P2]
│   ├── order_agent.py      [P2]
│   ├── product_agent.py    [P2]
│   ├── escalation_agent.py [P2]  Human handoff with full context transfer
│   ├── voice_agent.py      [P3]  STT → LLM → TTS pipeline
│   └── web_search_agent.py [P3]  Tavily / SerpAPI real-time search
│
├── tools/                  — LangChain @tool functions
│   ├── order_tool.py       [P2]
│   ├── product_tool.py     [P2]
│   ├── cart_tool.py        [P2]
│   ├── returns_tool.py     [P2]
│   ├── payment_tool.py     [P3]
│   ├── inventory_tool.py   [P3]
│   ├── crm_tool.py         [P3]
│   ├── web_search_tool.py  [P3]
│   ├── calculator_tool.py  [P3]
│   ├── notification_tool.py[P3]
│   └── tool_registry.py   [P2]  Registry with JSON schema validation
│
├── integrations/           — External API clients (one file per service)
│   ├── shopify_client.py   [P2]
│   ├── woocommerce_client.py[P2]
│   ├── cohere_client.py    [P2]
│   ├── stripe_client.py    [P3]
│   ├── razorpay_client.py  [P3]
│   ├── twilio_client.py    [P3]
│   ├── slack_client.py     [P3]
│   ├── teams_client.py     [P3]
│   ├── sendgrid_client.py  [P3]
│   ├── google_analytics_client.py [P3]
│   ├── deepgram_client.py  [P3]
│   └── elevenlabs_client.py[P3]
│
├── security/               — 10-file security layer
│   ├── auth.py             [MVP] JWT creation, decode, refresh, revocation
│   ├── rbac.py             [MVP] Role definitions & permission decorators
│   ├── input_validator.py  [MVP] Sanitise, max-length, regex guards
│   ├── prompt_guard.py     [MVP] Prompt injection detection
│   ├── cors_policy.py      [MVP]
│   ├── rate_limiter.py     [P2]  Redis-backed per-user limits (slowapi)
│   ├── pii_scrubber.py     [P2]  Strips email/phone/card numbers (presidio)
│   ├── encryption.py       [P2]  AES-256 + Fernet key management
│   ├── audit_logger.py     [P2]  Tamper-evident audit trail
│   └── secrets_manager.py  [P2]  AWS Secrets Manager / Vault client
│
├── database/               — SQLite (dev) / Postgres (prod)
│   ├── db_operations.py    [MVP] ORM CRUD: sessions, messages, feedback
│   ├── models.py           [P2]  Full ORM: User, Product, Order, Cart
│   ├── cache.py            [P2]  Redis get/set/invalidate with TTL
│   ├── migrations/         [P2]  Alembic migration scripts
│   └── chatbot.db          [MVP] SQLite dev database (auto-created)
│
├── routers/                — FastAPI APIRouter (one file per resource)
│   ├── chat.py             [MVP] POST /chat, GET /history, DELETE /session
│   ├── auth.py             [MVP] /login, /register, /refresh, /logout
│   ├── documents.py        [MVP] /upload, /docs, /doc/{id}
│   ├── feedback.py         [MVP] POST /feedback
│   ├── admin.py            [MVP] Prompt config, KB management, log viewer
│   ├── analytics.py        [P2]  KPI endpoints: latency, CSAT, deflection
│   ├── webhooks.py         [P3]  Shopify / Slack / Teams inbound webhooks
│   └── voice.py            [P3]  /voice/transcribe, /voice/speak
│
├── prompts/
│   ├── prompt_handler.py   [MVP] System prompt manager & runtime override
│   └── templates/
│       ├── system_prompt.txt      [MVP]
│       ├── rag_prompt.txt         [MVP]
│       ├── ecommerce_prompt.txt   [P2]
│       ├── escalation_prompt.txt  [P2]
│       └── voice_prompt.txt       [P3]
│
├── observability/
│   ├── logging_middleware.py [MVP] Structured JSON logs, request ID, PII masking
│   ├── health_checks.py    [MVP] /health/live, /health/ready, /health/startup
│   ├── metrics.py          [P2]  Prometheus counters, histograms, gauges
│   ├── tracing.py          [P2]  OpenTelemetry distributed tracing
│   ├── langsmith_tracer.py [P2]  LLM + RAG chain visibility
│   └── dashboards/
│       ├── grafana_chatbot.json [P2]
│       ├── prometheus.yml       [P2]
│       └── alerts.yml           [P2]  P95 > 3s, error rate > 1%, downtime
│
├── tests/                  — 4-tier test suite
│   ├── conftest.py         [MVP] Fixtures: in-memory DB, mock LLM, Redis mock
│   ├── unit/               [MVP] Utils, chatbot, DB, security, RAG, tools
│   ├── integration/        [MVP+P2] Chat API, document upload, agent graph, cache
│   ├── rag_eval/           [P2]  RAGAS: faithfulness, relevancy, context recall
│   ├── load/               [P2]  Locust: 100+ concurrent users
│   └── security_tests/     [P2]  OWASP, prompt injection, rate limiting
│
├── .github/workflows/
│   ├── ci.yml              [MVP] Lint → pytest → Docker build → push to ECR
│   ├── cd.yml              [P2]  Deploy to AWS ECS/Fargate on merge to main
│   └── security_scan.yml   [P2]  Snyk + Trivy vulnerability scan
│
├── infra/
│   ├── terraform/          [P2]  AWS ECS, RDS, S3, Secrets Manager, VPC
│   └── k8s/                [P3]  Kubernetes manifests for horizontal scale
│
├── frontend/src/
│   ├── ChatWidget.jsx      [MVP] React chat UI with RAG citation display
│   ├── AdminPanel.jsx      [MVP] Document upload, prompt editor, log viewer
│   ├── AnalyticsDashboard.jsx [P2]
│   └── VoiceInterface.jsx  [P3]  Microphone input + TTS audio playback
│
├── fine_tuning/            [P3]  LoRA/QLoRA domain fine-tuning
│   ├── prepare_dataset.py
│   ├── train_lora.py
│   ├── evaluate_model.py
│   └── push_to_hub.py
│
└── docs/
    ├── documentation.pdf
    ├── architecture_diagram.png
    └── api_reference.md
```

---

## Delivery Phases

| Phase | Timeline | Scope |
|-------|----------|-------|
| **MVP** | Weeks 1–6 | Core chat API, RAG pipeline, basic auth, SQLite, unit tests, CI |
| **P2** | Weeks 7–12 | LangGraph agents, eCommerce tools, Redis cache, load & security tests, CD |
| **P3** | Weeks 13–20 | Voice, fine-tuning, multi-cloud infra, Stripe/Razorpay, Twilio/Slack/Teams |

---

## Quick Start

### Prerequisites

- Python 3.11+
- Docker & Docker Compose
- Node.js 18+ (for frontend)

### 1. Clone & configure

```bash
git clone https://github.com/your-org/AI_Chatbot_System.git
cd AI_Chatbot_System
cp .env.example .env
# Edit .env with your API keys
```

### 2. Run with Docker Compose

```bash
docker-compose up --build
```

This starts: FastAPI server · Redis · Qdrant · Postgres · Grafana

### 3. Run locally (without Docker)

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app:app --reload
```

### 4. Frontend (dev)

```bash
cd frontend
npm install
npm run dev
```

API docs available at: `http://localhost:8000/docs`
Health check: `http://localhost:8000/health/live`

---

## Environment Variables

Copy `.env.example` to `.env` and fill in your values. Never commit `.env`.

```env
# LLM Providers
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Database
DATABASE_URL=postgresql://user:pass@localhost/chatbot
REDIS_URL=redis://localhost:6379

# Security
SECRET_KEY=your-jwt-secret
CORS_ORIGINS=http://localhost:3000

# Vector Store
QDRANT_URL=http://localhost:6333

# Observability (optional)
LANGCHAIN_API_KEY=ls__...
LANGCHAIN_TRACING_V2=true
```

Repository secrets for CI/CD: `OPENAI_API_KEY`, `DATABASE_URL`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`.

---

## API Reference

Full interactive docs at `/docs` (Swagger UI) or `/redoc`.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/chat` | Send a message; returns RAG-grounded response |
| `GET` | `/chat/history` | Retrieve session history |
| `DELETE` | `/chat/session` | Clear a session |
| `POST` | `/auth/register` | Register a new user |
| `POST` | `/auth/login` | Obtain JWT access token |
| `POST` | `/auth/refresh` | Refresh access token |
| `POST` | `/documents/upload` | Upload a document for RAG ingestion |
| `GET` | `/documents` | List indexed documents |
| `POST` | `/feedback` | Submit thumbs-up/down feedback |
| `GET` | `/health/live` | Liveness probe |
| `GET` | `/health/ready` | Readiness probe |

---

## Testing

```bash
# Unit tests (fast, no external deps)
pytest tests/unit/ -v

# Integration tests
pytest tests/integration/ -v

# RAG evaluation (RAGAS metrics)
pytest tests/rag_eval/ -v

# Load test (requires running server)
locust -f tests/load/locustfile.py --headless -u 100 -r 10

# Security tests
pytest tests/security_tests/ -v

# All tests with coverage
pytest --cov=. --cov-report=html
```

CI runs unit and integration tests on every pull request. Load and security tests run on merge to `main`.

---

## CI/CD

| Workflow | Trigger | Steps |
|----------|---------|-------|
| `ci.yml` | Every PR | Lint (ruff) → pytest → Docker build → push to ECR |
| `cd.yml` | Merge to `main` | Deploy to AWS ECS/Fargate |
| `security_scan.yml` | Daily + PR | Snyk + Trivy dependency & container scan |

**Branch protection** is enabled on `main`: PR review + CI passing required before merge.

---

## Contributing

1. Fork the repo and create a branch: `git checkout -b feat/your-feature`
2. Follow the [Naming Conventions](#naming-conventions) below
3. Write tests for any new functionality
4. Run `pytest` and `ruff check .` locally before pushing
5. Open a pull request against `main` with a clear description

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add Cohere reranker to RAG pipeline
fix: handle JWT expiry edge case in auth middleware
docs: update API reference for /voice endpoints
```

Issues are labelled `MVP`, `P2`, `P3`, `bug`, `enhancement`, and `documentation`.

---

## Naming Conventions

| Artefact | Convention | Example |
|----------|------------|---------|
| Python files | `snake_case.py` | `order_tool.py`, `db_operations.py` |
| Python classes | `PascalCase` | `OpenAIModel`, `DatabaseOperations` |
| Python functions | `snake_case` | `get_history()`, `build_prompt()` |
| Constants / env vars | `UPPER_SNAKE_CASE` | `OPENAI_API_KEY`, `MAX_HISTORY_TURNS` |
| React components | `PascalCase.jsx` | `ChatWidget.jsx`, `AdminPanel.jsx` |
| Test files | `test_<module>.py` | `test_database.py`, `test_rag.py` |
| Config / CI files | `kebab-case.yml` | `ci.yml`, `docker-compose.yml` |
| Directories | `snake_case/` | `rag/`, `fine_tuning/`, `security_tests/` |
| Git branches | `type/short-description` | `feat/rag-pipeline`, `fix/jwt-refresh` |

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*AI Chatbot System · v1.0 · 70 files · 15 modules · 3 delivery phases*
