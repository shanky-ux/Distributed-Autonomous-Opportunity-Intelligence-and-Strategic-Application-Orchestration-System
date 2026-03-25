<div align="center">

<br/>

```
██████╗  █████╗  ██████╗ ██╗███████╗ █████╗  ██████╗ ███████╗
██╔══██╗██╔══██╗██╔═══██╗██║██╔════╝██╔══██╗██╔═══██╗██╔════╝
██║  ██║███████║██║   ██║██║███████╗███████║██║   ██║███████╗
██║  ██║██╔══██║██║   ██║██║╚════██║██╔══██║██║   ██║╚════██║
██████╔╝██║  ██║╚██████╔╝██║███████║██║  ██║╚██████╔╝███████║
╚═════╝ ╚═╝  ╚═╝ ╚═════╝ ╚═╝╚══════╝╚═╝  ╚═╝ ╚═════╝ ╚══════╝
```

### **Distributed Autonomous Opportunity Intelligence and Strategic Application Orchestration Systems**

*Set it. Forget it. Get hired.*

<br/>

[![Python](https://img.shields.io/badge/Python_3.11+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI_0.111-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL_15+-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Redis](https://img.shields.io/badge/Redis_7+-DC382D?style=flat-square&logo=redis&logoColor=white)](https://redis.io)
[![OpenAI](https://img.shields.io/badge/GPT--4o-412991?style=flat-square&logo=openai&logoColor=white)](https://openai.com)
[![Celery](https://img.shields.io/badge/Celery_5.4-37814A?style=flat-square&logo=celery&logoColor=white)](https://docs.celeryq.dev)
[![Playwright](https://img.shields.io/badge/Playwright_1.44-2EAD33?style=flat-square&logo=playwright&logoColor=white)](https://playwright.dev)
[![ChromaDB](https://img.shields.io/badge/ChromaDB_0.5-FF6F00?style=flat-square)](https://trychroma.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

<br/>

> **An autonomous AI agent that wakes up every 6 hours, hunts down job opportunities across the internet,  
> scores them against your profile, writes tailored resumes & cover letters, submits applications,  
> and preps you for interviews — all while you sleep.**

<br/>

[**What It Does**](#-what-it-does) · [**How It Works**](#-how-it-works) · [**Agent Pipeline**](#-agent-pipeline) · [**Quick Start**](#-quick-start) · [**Configuration**](#-configuration) · [**API Reference**](#-api-reference) · [**Roadmap**](#-roadmap)

---

</div>

## ✦ What It Does

You configure your profile once. D.A.O.I.S.A.O.S handles the rest.

| Capability | Details |
|---|---|
| 🔍 **Multi-platform Scraping** | Continuously scans LinkedIn, Indeed, Internshala, and Wellfound every 6 hours |
| 🧠 **Dual-Model AI Scoring** | GPT-4o-mini for fast batch filtering · GPT-4o for deep match analysis |
| 📄 **ATS-Optimised Resumes** | Tailored per job with keyword injection, ATS score estimation, and PDF export |
| ✉️ **Cover Letter Generation** | Company-specific, role-aligned letters in configurable tone |
| 🤖 **Browser Automation** | Playwright bot submits applications across 6 ATS platforms |
| 🛡️ **Human-in-the-Loop Gate** | Telegram approval required before every submission (configurable) |
| 📊 **Full Lifecycle Tracking** | 14-state FSM with complete audit trail from discovery to offer |
| 🎤 **Interview Prep Engine** | Company intel, Q&A banks, AI mock sessions, Whisper audio analysis |
| 💬 **AI Chat Assistant** | "Apply to the top 3 AI internships in Europe" — and it does |
| 📈 **Market Intelligence** | Skill demand trends, salary bands, hiring velocity snapshot |

---

## ⚙️ How It Works

The platform runs as four cooperating layers: a Next.js dashboard, a FastAPI backend, a PostgreSQL + ChromaDB data layer, and a distributed Celery worker cluster — all orchestrated over Redis.

```
┌─────────────────────────────────────────────────────────────────┐
│  PRESENTATION LAYER   Next.js · TailwindCSS · JWT Auth          │
│  Job Feed · Resumes · App Tracker · Interview Prep · AI Chat    │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTPS / JWT Bearer
┌────────────────────────▼────────────────────────────────────────┐
│  APPLICATION LAYER   FastAPI · Uvicorn · SQLAlchemy · Pydantic  │
│  /auth  /jobs  /applications  /resumes  /agent  /analytics      │
└──────────┬──────────────────────────────┬───────────────────────┘
           │ asyncpg                      │ Celery .delay()
┌──────────▼──────────┐       ┌───────────▼───────────────────────┐
│    DATA LAYER        │       │        WORKER LAYER               │
│                      │       │                                   │
│  PostgreSQL 15+      │◄─────►│  scraping  ·  ai                 │
│  12 tables           │       │  automation  ·  notifications     │
│                      │       │                                   │
│  ChromaDB 0.5        │       │  Redis 7 (broker + backend)      │
│  RAG vector store    │       │  Beat scheduler · Flower UI      │
└─────────────────────┘       └───────────────────────────────────┘
```

---

## 🔄 Agent Pipeline

Every 6 hours, `run_main_agent_cycle` fires. Here's what happens:

```
Celery Beat fires every 6h
         │
         ▼
┌────────────────────────────────────────────────────────────┐
│  STAGE 1 · Job Discovery                   queue: scraping │
│                                                            │
│  LinkedIn ── Indeed ── Internshala ── Wellfound            │
│  Concurrent · fake-useragent rotation · dedup by ID        │
│  New jobs written → status: NEW                           │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  STAGE 2 · AI Analysis & Scoring              queue: ai   │
│                                                            │
│  Pass 1 → GPT-4o-mini    ~$0.001/job                      │
│           Batch filter · below threshold → SKIPPED        │
│                                                            │
│  Pass 2 → GPT-4o         structured JSON output           │
│           match_score · priority_score · skill_gaps       │
│           ats_keywords · salary estimates                  │
│           → JobAnalysis record · status: ANALYZED         │
└────────────────────────┬───────────────────────────────────┘
                         │  match_score ≥ threshold
                         ▼
┌────────────────────────────────────────────────────────────┐
│  STAGE 3 · Material Generation                queue: ai   │
│                                                            │
│  Resume Engine         Cover Letter Generator             │
│  ─────────────         ──────────────────────             │
│  ATS keyword inject    Company-specific context           │
│  ATS score estimate    Role alignment analysis            │
│  WeasyPrint PDF        Configurable tone                  │
│  Version tracking      Version tracking                   │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  STAGE 4 · Auto-Apply Bot            queue: automation    │
│                                                            │
│  ┌──────────────────────┐   ┌──────────────────────────┐  │
│  │ Telegram Gate        │──►│ Playwright Form Fill     │  │
│  │ Job + score preview  │   │ Workday · Greenhouse     │  │
│  │ Approve / Skip       │   │ Lever · LinkedIn         │  │
│  │ PENDING → QUEUED     │   │ Indeed · Internshala     │  │
│  └──────────────────────┘   │ CAPTCHA → escalation     │  │
│                              └──────────────────────────┘  │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  STAGE 5 · Tracking & Notifications   queue: notifications │
│                                                            │
│  PostgreSQL status update                                  │
│  ApplicationEvent audit log (every transition)            │
│  Telegram + email cycle digest                            │
└────────────────────────────────────────────────────────────┘
```

---

## 📬 Application Lifecycle

Every application flows through a **14-state FSM**. Every transition is recorded as an `ApplicationEvent` with timestamp, trigger source, and metadata.

```
                    PENDING_APPROVAL ──── user skips ────► SKIPPED
                           │
                    user approves
                           │
                        QUEUED
                           │
                        APPLYING ─── error / CAPTCHA ────► FAILED (retry ×3)
                           │
                        APPLIED
                           │
                        VIEWED ──── not progressed ──────► REJECTED
                           │
                      SHORTLISTED
                           │
                  INTERVIEW_SCHEDULED
                           │
                  INTERVIEW_COMPLETED
                      /          \
              OFFER_RECEIVED    REJECTED
              /          \
    OFFER_ACCEPTED   OFFER_DECLINED

    ── any state ──► WITHDRAWN (user-initiated)
```

`ApplicationMethod`: `AUTO_BOT` · `EASY_APPLY` · `MANUAL` · `EMAIL`

---

## 🚀 Quick Start

### Prerequisites

```
Python 3.11+   PostgreSQL 15+   Redis 7+   Node.js 18+   Chromium (via Playwright)
```

### 1 · Clone

```bash
git clone https://github.com/Sudharsanselvaraj/Distributed-Autonomous-Opportunity-Intelligence-and-Strategic-Application-Orchestration-System.git
cd Distributed-Autonomous-Opportunity-Intelligence-and-Strategic-Application-Orchestration-System
```

### 2 · Python environment

```bash
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
playwright install chromium
```

### 3 · Configure

```bash
cp career_platform/.env.example career_platform/.env
# Fill in your profile, API keys, and credentials — see Configuration below
```

### 4 · Database

```bash
psql -U postgres -c "CREATE DATABASE career_platform;"
cd career_platform && alembic upgrade head
```

### 5 · Launch

**Windows (one command):**
```bat
setup.bat           # first-time setup
start_platform.bat  # start everything
```

**Linux / macOS (four terminals):**
```bash
# 1 — API server
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# 2 — Celery workers
celery -A app.agents.tasks.celery_app worker --loglevel=info \
  -Q scraping,ai,automation,notifications --concurrency=4

# 3 — Beat scheduler
celery -A app.agents.tasks.celery_app beat --loglevel=info

# 4 — Flower dashboard (optional)
celery -A app.agents.tasks.celery_app flower --port=5555
```

### 6 · Register & authenticate

```bash
# Create account
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "secret", "full_name": "Your Name"}'

# Get JWT token
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "secret"}'
```

| Service | URL |
|---|---|
| Swagger UI | `http://localhost:8000/api/docs` |
| ReDoc | `http://localhost:8000/api/redoc` |
| Health check | `http://localhost:8000/health` |
| Flower | `http://localhost:5555` |

---

## ⚙️ Configuration

All settings live in `career_platform/.env`. A complete template is in `.env.example`.

### Your profile — the most important section

Everything the AI generates is grounded in this. Fill it in carefully.

```env
USER_NAME="Your Name"
USER_EMAIL=you@example.com
USER_PHONE=+91-XXXXXXXXXX
USER_LOCATION="Chennai, India"
USER_DESIRED_ROLES=["Computer Vision Engineer","ML Engineer","AI Research Intern","Data Scientist"]
USER_DESIRED_LOCATIONS=["Remote","Bangalore","Hyderabad","Chennai","Europe","USA"]
USER_EXPERIENCE_LEVEL=entry          # entry | mid | senior
USER_OPEN_TO_REMOTE=true
USER_MIN_SALARY=0
```

### AI layer

```env
OPENAI_API_KEY=sk-...
OPENAI_MODEL_HEAVY=gpt-4o            # deep analysis, resume/cover letter, interview prep
OPENAI_MODEL_LIGHT=gpt-4o-mini       # batch scoring, filtering, chat
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
```

### Auto-apply controls

> ⚠️ Auto-apply is **disabled by default**. Review these before enabling.

```env
AUTO_APPLY_ENABLED=false
AUTO_APPLY_MATCH_THRESHOLD=75        # only queue jobs scoring ≥ this
AUTO_APPLY_DAILY_LIMIT=10            # hard cap per calendar day
AUTO_APPLY_REQUIRE_APPROVAL=true     # Telegram approval before each submission
```

### Notifications

```env
TELEGRAM_BOT_TOKEN=...               # create via @BotFather
TELEGRAM_CHAT_ID=...                 # get via @userinfobot
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=you@gmail.com
SMTP_PASSWORD=your-app-password      # Gmail App Password, not account password
```

---

## 🌐 API Reference

Full interactive docs at `/api/docs` (Swagger) and `/api/redoc` (ReDoc).  
All endpoints require `Authorization: Bearer <token>`.

### Routers

| Prefix | Key Endpoints |
|---|---|
| `/api/auth` | `POST /register` · `POST /login` · `GET /me` |
| `/api/jobs` | `GET /` (paginated + filterable) · `POST /{id}/analyze` · `POST /{id}/skip` |
| `/api/applications` | `GET /` · `PATCH /{id}/status` · `GET /stats` |
| `/api/resumes` | `POST /generate` · `GET /{id}/download` |
| `/api/agent` | `POST /run` · `POST /cycle/trigger` · `GET /tasks` |
| `/api/analytics` | `GET /dashboard` · `GET /market` · `GET /skill-gaps` |
| `/api/chat` | `POST /message` · `GET /history` |

### Example calls

```bash
# Trigger a manual scrape cycle
curl -X POST http://localhost:8000/api/agent/run \
  -H "Authorization: Bearer <token>" \
  -d '{"task": "scrape_jobs", "params": {"platforms": ["linkedin", "indeed"]}}'

# Top remote jobs with match score ≥ 80
curl "http://localhost:8000/api/jobs?min_match_score=80&work_mode=remote&sort_by=match_score" \
  -H "Authorization: Bearer <token>"

# Generate a tailored resume for a specific job
curl -X POST http://localhost:8000/api/resumes/generate \
  -H "Authorization: Bearer <token>" \
  -d '{"job_id": "<uuid>", "base_resume_id": "<uuid>"}'

# Natural language command to the AI assistant
curl -X POST http://localhost:8000/api/chat/message \
  -H "Authorization: Bearer <token>" \
  -d '{"message": "Find AI internships in Europe and apply to the top 3 matches"}'
```

---

## 🗄️ Database Schema

Twelve tables across four domains:

```
users ──1:1──► user_profiles
     └──1:N──► user_skills
     └──1:N──► resumes ──────────────────────────────────┐
                                                          │
jobs ──1:1──► job_analyses                               │
     └──1:N──► applications ◄── resume_id ───────────────┘
                    └──1:N──► application_events
                    └──1:N──► interviews
                                   └──1:N──► mock_interview_sessions

+ agent_tasks · audit_logs · credential_vaults · market_snapshots
```

### Enumerations

```python
JobSource:    linkedin | indeed | internshala | wellfound | glassdoor | company_site | manual
JobType:      full_time | part_time | internship | contract | freelance
WorkMode:     remote | onsite | hybrid | unknown
JobStatus:    new | analyzed | queued | applied | skipped | expired
Seniority:    entry | mid | senior | lead | unknown
RoleCategory: computer_vision | nlp | mlops | data_science | software_engineering | other
Difficulty:   easy | medium | hard
InterviewType: phone_screen | technical | behavioral | system_design | hr | final | take_home
```

---

## 🎤 Interview Preparation Engine

When an application advances to `INTERVIEW_SCHEDULED`, the system automatically generates a full prep package:

**Company Intel Report** — products, recent news, known tech stack, culture signals, Glassdoor rating, interview format  
**Question Bank** — technical questions with expected answers + difficulty ratings, behavioural questions with STAR hints  
**Mock Interview Sessions** — full transcript with AI interviewer, scored debrief covering technical depth, communication, and confidence  
**Recording Analysis** — Whisper transcription + GPT-4o analysis of filler words, clarity, pacing, and confidence

---

## 🔒 Security

| Concern | Approach |
|---|---|
| **Auth** | JWT Bearer tokens · bcrypt password hashing · 24h expiry |
| **Credentials at rest** | AES-256-GCM · PBKDF2-HMAC-SHA256 (100k iterations) |
| **GDPR** | `UserConsent` model · full data export · 30-day deletion window |
| **Audit trail** | All sensitive ops logged to `AuditLog` table |
| **Auto-apply safety** | Approval gate + daily limit + match threshold — three independent guards |

```bash
# Generate secure keys
openssl rand -hex 32
```

---

## 🗂️ Project Structure

```
career_platform/
├── app/
│   ├── agents/
│   │   ├── apply_bot.py          # Playwright auto-apply; ATS detection and form fill
│   │   ├── tasks.py              # Celery tasks, Beat schedule, 4-queue routing
│   │   └── scrapers/
│   │       ├── base.py           # Abstract base: rate limiting, dedup, persistence
│   │       ├── linkedin.py
│   │       ├── indeed.py
│   │       ├── internshala.py
│   │       └── wellfound.py
│   ├── api/routes/               # Auth, jobs, applications, resumes, agent, analytics, chat
│   ├── core/
│   │   ├── config.py             # Pydantic Settings — .env loading
│   │   └── database.py           # Async SQLAlchemy engine and session factory
│   ├── models/                   # SQLAlchemy ORM models (job, application, user, interview)
│   ├── services/
│   │   ├── job_analyzer.py       # Two-pass LLM pipeline
│   │   ├── resume_service.py     # ATS tailoring + PDF rendering
│   │   ├── cover_letter_service.py
│   │   ├── interview_service.py  # Prep package generation + Whisper analysis
│   │   ├── ai_assistant.py       # Chat: intent detection + agent dispatch
│   │   ├── notification_service.py
│   │   ├── follow_up_service.py
│   │   └── market_service.py
│   ├── schemas/                  # Pydantic v2 request/response schemas
│   └── main.py                   # FastAPI app factory + lifespan
├── alembic/                      # Database migrations
├── .env.example                  # Full configuration template
└── requirements.txt              # 40+ pinned dependencies
```

---

## 🗓️ Automation Schedule

No cron needed — all tasks are registered with Celery Beat.

| Task | Interval | Description |
|---|---|---|
| `run_main_agent_cycle` | Every 6 hours | Full pipeline: discover → analyse → generate → apply → notify |
| `check_follow_ups` | Every hour | 7-day recruiter follow-ups, interview confirmations, thank-you emails |
| `take_market_snapshot` | Daily | Skill demand, salary trends, hiring velocity |
| `update_resume_performance` | Every 6 hours | Response rate recalculation per resume version |

**Queue rate limits:** `scraping/LinkedIn` 10/min · `scraping/Indeed` 20/min · `automation` 5/min  
**Retry policy:** max 3 retries · exponential backoff · `task_acks_late = true`

---

## 🛠️ Tech Stack

<details>
<summary>Full stack (click to expand)</summary>

| Layer | Technology | Version |
|---|---|---|
| Web framework | FastAPI | 0.111.0 |
| ASGI server | Uvicorn | 0.30.1 |
| ORM | SQLAlchemy (async) | 2.0.30 |
| Database | PostgreSQL | 15+ |
| Migrations | Alembic | 1.13.1 |
| Task queue | Celery | 5.4.0 |
| Message broker | Redis | 7+ |
| Queue monitor | Flower | 2.0.1 |
| LLM (heavy) | OpenAI GPT-4o | openai 1.30.1 |
| LLM (light) | OpenAI GPT-4o-mini | openai 1.30.1 |
| Embeddings | text-embedding-3-small | openai 1.30.1 |
| Audio | OpenAI Whisper | openai-whisper 20231117 |
| LLM orchestration | LangChain | 0.2.1 |
| Vector store | ChromaDB | 0.5.0 |
| Browser automation | Playwright | 1.44.0 |
| Async HTTP | httpx + aiohttp | 0.27.0 / 3.9.5 |
| HTML parsing | BeautifulSoup4 + lxml | 4.12.3 / 5.2.2 |
| Telegram | python-telegram-bot | 21.2 |
| Email | aiosmtplib + Jinja2 | 3.0.1 / 3.1.4 |
| PDF generation | WeasyPrint + ReportLab | 62.1 / 4.2.0 |
| JWT | python-jose | 3.3.0 |
| Password hashing | passlib + bcrypt | 1.7.4 / 4.1.3 |
| Validation | Pydantic v2 | 2.7.1 |
| Retry logic | tenacity | 8.3.0 |
| User agent rotation | fake-useragent | 1.5.1 |
| Frontend | Next.js + TailwindCSS | 18+ / 3+ |

</details>

---

## 🗺️ Roadmap

- [ ] Glassdoor scraper integration
- [ ] Docker Compose deployment configuration
- [ ] S3 storage backend for resumes and recordings
- [ ] Frontend Next.js dashboard (job feed, tracker, interview prep)
- [ ] Salary negotiation assistant powered by market snapshots
- [ ] Multi-user support with per-user agent isolation
- [ ] Webhook support for ATS status callbacks (Greenhouse, Lever)
- [ ] LangSmith tracing for LLM observability
- [ ] Full test suite coverage for agent pipeline and services

---

## 🤝 Contributing

Contributions are welcome. Please open an issue to discuss proposed changes before submitting a pull request.  
All new code should include type annotations and pass the existing test suite.

---

## 📄 License

[MIT License](LICENSE) — do whatever you want, just don't blame me if you get too many job offers.

---

<div align="center">

Built with obsessive attention to detail by [Sudharsan Selvaraj](https://github.com/Sudharsanselvaraj)

*Because applying to jobs manually is a solved problem.*

</div>
