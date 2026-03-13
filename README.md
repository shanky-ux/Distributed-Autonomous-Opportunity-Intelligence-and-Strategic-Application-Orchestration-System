<div align="center">

# D.A.O.I.S.A.O.S

### Distributed Autonomous Opportunity Intelligence and Strategic Application Orchestration System

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Redis](https://img.shields.io/badge/Redis-7+-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com)
[![Celery](https://img.shields.io/badge/Celery-5.4-37814A?style=for-the-badge&logo=celery&logoColor=white)](https://docs.celeryq.dev)
[![Playwright](https://img.shields.io/badge/Playwright-1.44-2EAD33?style=for-the-badge&logo=playwright&logoColor=white)](https://playwright.dev)
[![ChromaDB](https://img.shields.io/badge/ChromaDB-0.5-FF6F00?style=for-the-badge)](https://trychroma.com)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

**A personal AI career automation platform that autonomously discovers job opportunities, generates tailored application materials, submits applications through browser automation, tracks the full application lifecycle, and orchestrates interview preparation — all through a distributed background agent system.**

[Overview](#overview) · [Architecture](#system-architecture) · [Agent Pipeline](#agent-pipeline) · [Core Modules](#core-modules) · [Data Models](#data-models) · [Quick Start](#quick-start) · [Configuration](#configuration) · [API Reference](#api-reference) · [Automation Schedule](#automation-schedule) · [Tech Stack](#tech-stack) · [Security](#security) · [Roadmap](#roadmap)

</div>

---

## Overview

D.A.O.I.S.A.O.S is a **distributed, AI-powered career automation platform** designed for single-user personal deployment. Once configured, it operates entirely in the background — continuously scraping job boards across multiple platforms, scoring each opportunity against your career profile through a dual-model LLM pipeline, generating ATS-optimised resumes and cover letters, submitting applications via a Playwright browser agent, and delivering real-time notifications through Telegram and email.

The platform is built around three core principles:

**Autonomy** — The platform runs on a self-scheduling Celery Beat loop. Every decision point (scrape, score, generate, submit) is fully automated and logged as an auditable `AgentTask` record with exponential-backoff retry on failure.

**Intelligence** — Two OpenAI models operate in tandem. `GPT-4o-mini` handles high-volume, low-cost operations: batch job filtering, match scoring, and chat responses. `GPT-4o` is reserved for deep, high-value work: full job description analysis, resume tailoring, cover letter generation, and interview preparation. A ChromaDB vector store holds semantic embeddings of your profile, past applications, and recruiter interactions, enabling RAG-based personalisation across all AI calls.

**Safety** — Auto-apply is disabled by default. When enabled, every application first passes through a configurable human-in-the-loop gate: a Telegram notification is dispatched to the user, and the bot waits for explicit approval before submitting. A daily application limit and a minimum match-score threshold provide additional safeguards.

---

## System Architecture

The platform is composed of four layers: a Next.js presentation layer, a FastAPI application layer, a combined data layer (PostgreSQL + ChromaDB), and a distributed worker layer powered by Celery and Redis. All long-running tasks are offloaded from the request cycle to purpose-specific Celery queues.

<div align="center">

<svg width="860" viewBox="0 0 860 900" xmlns="http://www.w3.org/2000/svg" font-family="ui-monospace,SFMono-Regular,Menlo,monospace" font-size="12">
  <defs>
    <marker id="a1" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#6366f1" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
    <marker id="a2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#64748b" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  <!-- PRESENTATION LAYER -->
  <rect x="20" y="20" width="820" height="130" rx="12" fill="#f0f0ff" stroke="#818cf8" stroke-width="1.5"/>
  <text x="36" y="44" font-weight="700" font-size="13" fill="#3730a3">PRESENTATION LAYER</text>
  <text x="36" y="60" font-size="11" fill="#6366f1">Next.js · TailwindCSS · HTTPS / JWT Bearer</text>
  <rect x="40" y="70" width="130" height="60" rx="8" fill="#e0e7ff" stroke="#818cf8" stroke-width="1"/>
  <text x="105" y="97" text-anchor="middle" font-weight="600" fill="#3730a3">Job Feed</text>
  <text x="105" y="115" text-anchor="middle" font-size="10" fill="#6366f1">Browse &amp; filter</text>
  <rect x="185" y="70" width="130" height="60" rx="8" fill="#e0e7ff" stroke="#818cf8" stroke-width="1"/>
  <text x="250" y="97" text-anchor="middle" font-weight="600" fill="#3730a3">Resumes</text>
  <text x="250" y="115" text-anchor="middle" font-size="10" fill="#6366f1">Generate &amp; download</text>
  <rect x="330" y="70" width="130" height="60" rx="8" fill="#e0e7ff" stroke="#818cf8" stroke-width="1"/>
  <text x="395" y="97" text-anchor="middle" font-weight="600" fill="#3730a3">App Tracker</text>
  <text x="395" y="115" text-anchor="middle" font-size="10" fill="#6366f1">14-state FSM view</text>
  <rect x="475" y="70" width="130" height="60" rx="8" fill="#e0e7ff" stroke="#818cf8" stroke-width="1"/>
  <text x="540" y="97" text-anchor="middle" font-weight="600" fill="#3730a3">Interview Prep</text>
  <text x="540" y="115" text-anchor="middle" font-size="10" fill="#6366f1">Mock sessions</text>
  <rect x="620" y="70" width="200" height="60" rx="8" fill="#e0e7ff" stroke="#818cf8" stroke-width="1"/>
  <text x="720" y="97" text-anchor="middle" font-weight="600" fill="#3730a3">AI Chat Assistant</text>
  <text x="720" y="115" text-anchor="middle" font-size="10" fill="#6366f1">Natural language commands</text>

  <!-- Arrow down -->
  <line x1="430" y1="150" x2="430" y2="178" stroke="#6366f1" stroke-width="1.5" marker-end="url(#a1)"/>
  <text x="445" y="169" font-size="10" fill="#6366f1">HTTPS / JWT Bearer</text>

  <!-- APPLICATION LAYER -->
  <rect x="20" y="180" width="820" height="110" rx="12" fill="#eff6ff" stroke="#60a5fa" stroke-width="1.5"/>
  <text x="36" y="204" font-weight="700" font-size="13" fill="#1d4ed8">APPLICATION LAYER</text>
  <text x="36" y="220" font-size="11" fill="#3b82f6">FastAPI · Uvicorn · SQLAlchemy asyncpg · Pydantic v2</text>
  <rect x="40" y="232" width="90" height="40" rx="6" fill="#dbeafe" stroke="#60a5fa" stroke-width="1"/>
  <text x="85" y="257" text-anchor="middle" font-weight="600" font-size="11" fill="#1d4ed8">/auth</text>
  <rect x="140" y="232" width="90" height="40" rx="6" fill="#dbeafe" stroke="#60a5fa" stroke-width="1"/>
  <text x="185" y="257" text-anchor="middle" font-weight="600" font-size="11" fill="#1d4ed8">/jobs</text>
  <rect x="240" y="232" width="130" height="40" rx="6" fill="#dbeafe" stroke="#60a5fa" stroke-width="1"/>
  <text x="305" y="257" text-anchor="middle" font-weight="600" font-size="11" fill="#1d4ed8">/applications</text>
  <rect x="380" y="232" width="110" height="40" rx="6" fill="#dbeafe" stroke="#60a5fa" stroke-width="1"/>
  <text x="435" y="257" text-anchor="middle" font-weight="600" font-size="11" fill="#1d4ed8">/resumes</text>
  <rect x="500" y="232" width="90" height="40" rx="6" fill="#dbeafe" stroke="#60a5fa" stroke-width="1"/>
  <text x="545" y="257" text-anchor="middle" font-weight="600" font-size="11" fill="#1d4ed8">/agent</text>
  <rect x="600" y="232" width="110" height="40" rx="6" fill="#dbeafe" stroke="#60a5fa" stroke-width="1"/>
  <text x="655" y="257" text-anchor="middle" font-weight="600" font-size="11" fill="#1d4ed8">/analytics</text>
  <rect x="720" y="232" width="100" height="40" rx="6" fill="#dbeafe" stroke="#60a5fa" stroke-width="1"/>
  <text x="770" y="257" text-anchor="middle" font-weight="600" font-size="11" fill="#1d4ed8">/chat</text>

  <!-- Arrows down from app layer -->
  <line x1="230" y1="290" x2="230" y2="318" stroke="#64748b" stroke-width="1.5" marker-end="url(#a2)"/>
  <text x="120" y="310" font-size="10" fill="#64748b">SQLAlchemy (asyncpg)</text>
  <line x1="620" y1="290" x2="620" y2="318" stroke="#64748b" stroke-width="1.5" marker-end="url(#a2)"/>
  <text x="540" y="310" font-size="10" fill="#64748b">Celery .delay()</text>

  <!-- DATA LAYER -->
  <rect x="20" y="320" width="380" height="250" rx="12" fill="#f0fdf4" stroke="#4ade80" stroke-width="1.5"/>
  <text x="36" y="344" font-weight="700" font-size="13" fill="#15803d">DATA LAYER</text>

  <rect x="40" y="355" width="340" height="100" rx="8" fill="#dcfce7" stroke="#4ade80" stroke-width="1"/>
  <text x="210" y="378" text-anchor="middle" font-weight="600" font-size="12" fill="#15803d">PostgreSQL 15+</text>
  <text x="210" y="397" text-anchor="middle" font-size="10" fill="#166534">users · jobs · job_analyses · resumes</text>
  <text x="210" y="413" text-anchor="middle" font-size="10" fill="#166534">applications · application_events</text>
  <text x="210" y="429" text-anchor="middle" font-size="10" fill="#166534">interviews · agent_tasks · audit_logs</text>
  <text x="210" y="447" text-anchor="middle" font-size="10" fill="#166534">credential_vaults · market_snapshots</text>

  <rect x="40" y="468" width="340" height="82" rx="8" fill="#dcfce7" stroke="#4ade80" stroke-width="1"/>
  <text x="210" y="491" text-anchor="middle" font-weight="600" font-size="12" fill="#15803d">ChromaDB 0.5 — Vector Store</text>
  <text x="210" y="510" text-anchor="middle" font-size="10" fill="#166534">user_profile embeddings</text>
  <text x="210" y="526" text-anchor="middle" font-size="10" fill="#166534">jobs · resumes · recruiter interactions</text>
  <text x="210" y="542" text-anchor="middle" font-size="10" fill="#166534">RAG retrieval for all AI calls</text>

  <!-- WORKER LAYER -->
  <rect x="440" y="320" width="400" height="250" rx="12" fill="#fff7ed" stroke="#fb923c" stroke-width="1.5"/>
  <text x="456" y="344" font-weight="700" font-size="13" fill="#c2410c">WORKER LAYER</text>

  <rect x="458" y="355" width="364" height="140" rx="8" fill="#ffedd5" stroke="#fb923c" stroke-width="1"/>
  <text x="640" y="378" text-anchor="middle" font-weight="600" font-size="12" fill="#c2410c">Celery Workers · 4 Queues</text>
  <text x="470" y="400" font-size="10" fill="#9a3412">scraping    LinkedIn · Indeed · Internshala · Wellfound</text>
  <text x="470" y="418" font-size="10" fill="#9a3412">ai              Job analysis · Resume · Cover letter gen</text>
  <text x="470" y="436" font-size="10" fill="#9a3412">automation  Playwright ApplyBot · CAPTCHA handling</text>
  <text x="470" y="454" font-size="10" fill="#9a3412">notifications  Telegram bot · SMTP email digest</text>
  <text x="470" y="472" font-size="10" fill="#9a3412">Beat scheduler — 6h cycle · 1h follow-ups · daily market</text>

  <rect x="458" y="508" width="364" height="42" rx="8" fill="#ffedd5" stroke="#fb923c" stroke-width="1"/>
  <text x="640" y="526" text-anchor="middle" font-weight="600" font-size="12" fill="#c2410c">Redis 7</text>
  <text x="640" y="542" text-anchor="middle" font-size="10" fill="#9a3412">Broker (db 0) · Result backend (db 1)</text>

  <!-- Arrow between data and worker -->
  <line x1="420" y1="445" x2="438" y2="445" stroke="#64748b" stroke-width="1.5" marker-end="url(#a2)"/>

  <!-- EXTERNAL SERVICES -->
  <rect x="20" y="600" width="820" height="280" rx="12" fill="#fefce8" stroke="#facc15" stroke-width="1.5"/>
  <text x="36" y="624" font-weight="700" font-size="13" fill="#854d0e">EXTERNAL SERVICES</text>

  <rect x="40" y="636" width="240" height="224" rx="8" fill="#fef9c3" stroke="#facc15" stroke-width="1"/>
  <text x="160" y="659" text-anchor="middle" font-weight="600" font-size="12" fill="#854d0e">OpenAI API</text>
  <text x="160" y="678" text-anchor="middle" font-size="10" fill="#713f12">GPT-4o — deep analysis</text>
  <text x="160" y="696" text-anchor="middle" font-size="10" fill="#713f12">resume &amp; cover letter generation</text>
  <text x="160" y="714" text-anchor="middle" font-size="10" fill="#713f12">interview prep + scoring</text>
  <text x="160" y="732" text-anchor="middle" font-size="10" fill="#713f12">GPT-4o-mini — batch scoring</text>
  <text x="160" y="750" text-anchor="middle" font-size="10" fill="#713f12">quick filter · chat responses</text>
  <text x="160" y="768" text-anchor="middle" font-size="10" fill="#713f12">text-embedding-3-small — vectors</text>
  <text x="160" y="786" text-anchor="middle" font-size="10" fill="#713f12">Whisper — audio transcription</text>
  <text x="160" y="804" text-anchor="middle" font-size="10" fill="#713f12">LangChain · RAG orchestration</text>

  <rect x="300" y="636" width="240" height="224" rx="8" fill="#fef9c3" stroke="#facc15" stroke-width="1"/>
  <text x="420" y="659" text-anchor="middle" font-weight="600" font-size="12" fill="#854d0e">Notifications</text>
  <text x="420" y="678" text-anchor="middle" font-size="10" fill="#713f12">Telegram Bot API</text>
  <text x="420" y="696" text-anchor="middle" font-size="10" fill="#713f12">Inline approve / skip keyboards</text>
  <text x="420" y="714" text-anchor="middle" font-size="10" fill="#713f12">Human-in-the-loop gate</text>
  <text x="420" y="732" text-anchor="middle" font-size="10" fill="#713f12">CAPTCHA escalation alerts</text>
  <text x="420" y="750" text-anchor="middle" font-size="10" fill="#713f12">Cycle result digests</text>
  <text x="420" y="768" text-anchor="middle" font-size="10" fill="#713f12">SMTP (Gmail / custom)</text>
  <text x="420" y="786" text-anchor="middle" font-size="10" fill="#713f12">Jinja2 HTML email templates</text>
  <text x="420" y="804" text-anchor="middle" font-size="10" fill="#713f12">Follow-up automation</text>

  <rect x="560" y="636" width="260" height="224" rx="8" fill="#fef9c3" stroke="#facc15" stroke-width="1"/>
  <text x="690" y="659" text-anchor="middle" font-weight="600" font-size="12" fill="#854d0e">Job Platforms</text>
  <text x="690" y="678" text-anchor="middle" font-size="10" fill="#713f12">LinkedIn — authenticated scraper</text>
  <text x="690" y="696" text-anchor="middle" font-size="10" fill="#713f12">Indeed — search parser</text>
  <text x="690" y="714" text-anchor="middle" font-size="10" fill="#713f12">Internshala — internships</text>
  <text x="690" y="732" text-anchor="middle" font-size="10" fill="#713f12">Wellfound — startup jobs</text>
  <text x="690" y="750" text-anchor="middle" font-size="10" fill="#713f12">Glassdoor (roadmap)</text>
  <text x="690" y="768" text-anchor="middle" font-size="10" fill="#713f12">ATS: Workday · Greenhouse</text>
  <text x="690" y="786" text-anchor="middle" font-size="10" fill="#713f12">ATS: Lever · LinkedIn Easy Apply</text>
  <text x="690" y="804" text-anchor="middle" font-size="10" fill="#713f12">ATS: Indeed · Internshala</text>

  <!-- Arrows to external -->
  <line x1="230" y1="570" x2="230" y2="598" stroke="#64748b" stroke-width="1.5" marker-end="url(#a2)"/>
  <line x1="620" y1="570" x2="620" y2="598" stroke="#64748b" stroke-width="1.5" marker-end="url(#a2)"/>
</svg>

</div>

---

## Agent Pipeline

The central automation cycle runs every 6 hours via `run_main_agent_cycle`. Each stage is a discrete Celery task routed to a purpose-specific queue with full `AgentTask` audit logging and exponential-backoff retry on failure.

<div align="center">

<svg width="860" viewBox="0 0 860 860" xmlns="http://www.w3.org/2000/svg" font-family="ui-monospace,SFMono-Regular,Menlo,monospace" font-size="12">
  <defs>
    <marker id="ap" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#64748b" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  <!-- Trigger -->
  <rect x="280" y="14" width="300" height="40" rx="8" fill="#f1f5f9" stroke="#94a3b8" stroke-width="1.5"/>
  <text x="430" y="39" text-anchor="middle" font-weight="700" font-size="12" fill="#334155">Celery Beat — every 6 hours</text>
  <line x1="430" y1="54" x2="430" y2="76" stroke="#64748b" stroke-width="1.5" marker-end="url(#ap)"/>

  <!-- STAGE 1 -->
  <rect x="20" y="78" width="820" height="118" rx="10" fill="#ecfdf5" stroke="#34d399" stroke-width="1.5"/>
  <text x="36" y="100" font-weight="700" font-size="13" fill="#065f46">Stage 1 — Job Discovery</text>
  <text x="36" y="116" font-size="10" fill="#059669">queue: scraping · concurrent execution · fake-useragent rotation · dedup by source_job_id</text>
  <rect x="36" y="124" width="155" height="56" rx="7" fill="#d1fae5" stroke="#34d399" stroke-width="1"/>
  <text x="113" y="147" text-anchor="middle" font-weight="600" font-size="11" fill="#065f46">LinkedIn</text>
  <text x="113" y="165" text-anchor="middle" font-size="10" fill="#065f46">Session auth · Easy Apply</text>
  <rect x="205" y="124" width="155" height="56" rx="7" fill="#d1fae5" stroke="#34d399" stroke-width="1"/>
  <text x="282" y="147" text-anchor="middle" font-weight="600" font-size="11" fill="#065f46">Indeed</text>
  <text x="282" y="165" text-anchor="middle" font-size="10" fill="#065f46">Search · optional creds</text>
  <rect x="374" y="124" width="155" height="56" rx="7" fill="#d1fae5" stroke="#34d399" stroke-width="1"/>
  <text x="451" y="147" text-anchor="middle" font-weight="600" font-size="11" fill="#065f46">Internshala</text>
  <text x="451" y="165" text-anchor="middle" font-size="10" fill="#065f46">Internships · full-time</text>
  <rect x="543" y="124" width="155" height="56" rx="7" fill="#d1fae5" stroke="#34d399" stroke-width="1"/>
  <text x="620" y="147" text-anchor="middle" font-weight="600" font-size="11" fill="#065f46">Wellfound</text>
  <text x="620" y="165" text-anchor="middle" font-size="10" fill="#065f46">Startup jobs · session</text>
  <rect x="712" y="124" width="112" height="56" rx="7" fill="#d1fae5" stroke="#34d399" stroke-width="1"/>
  <text x="768" y="147" text-anchor="middle" font-weight="600" font-size="11" fill="#065f46">Dedup</text>
  <text x="768" y="165" text-anchor="middle" font-size="10" fill="#065f46">→ status = NEW</text>
  <line x1="430" y1="196" x2="430" y2="218" stroke="#64748b" stroke-width="1.5" marker-end="url(#ap)"/>

  <!-- STAGE 2 -->
  <rect x="20" y="220" width="820" height="152" rx="10" fill="#eff6ff" stroke="#60a5fa" stroke-width="1.5"/>
  <text x="36" y="242" font-weight="700" font-size="13" fill="#1e40af">Stage 2 — AI Analysis &amp; Scoring</text>
  <text x="36" y="258" font-size="10" fill="#3b82f6">queue: ai · dual-model LLM pipeline · RAG-enhanced via ChromaDB</text>
  <rect x="36" y="266" width="368" height="88" rx="7" fill="#dbeafe" stroke="#60a5fa" stroke-width="1"/>
  <text x="220" y="288" text-anchor="middle" font-weight="600" font-size="11" fill="#1e40af">Pass 1 — GPT-4o-mini  (batch filter, ~$0.001/job)</text>
  <text x="220" y="307" text-anchor="middle" font-size="10" fill="#1e3a8a">Input: title + description + USER_DESIRED_ROLES</text>
  <text x="220" y="323" text-anchor="middle" font-size="10" fill="#1e3a8a">Output: preliminary_score (0–100) · is_relevant</text>
  <text x="220" y="341" text-anchor="middle" font-size="10" fill="#1e3a8a">Below threshold → SKIPPED · exit pipeline</text>
  <rect x="420" y="266" width="400" height="88" rx="7" fill="#dbeafe" stroke="#60a5fa" stroke-width="1"/>
  <text x="620" y="288" text-anchor="middle" font-weight="600" font-size="11" fill="#1e40af">Pass 2 — GPT-4o  (deep analysis, structured JSON)</text>
  <text x="620" y="307" text-anchor="middle" font-size="10" fill="#1e3a8a">match_score · priority_score · skill_gaps</text>
  <text x="620" y="323" text-anchor="middle" font-size="10" fill="#1e3a8a">ats_keywords · seniority · salary estimates</text>
  <text x="620" y="341" text-anchor="middle" font-size="10" fill="#1e3a8a">→ JobAnalysis record · status = ANALYZED</text>
  <line x1="404" y1="310" x2="418" y2="310" stroke="#64748b" stroke-width="1.5" marker-end="url(#ap)"/>
  <line x1="430" y1="372" x2="430" y2="394" stroke="#64748b" stroke-width="1.5" marker-end="url(#ap)"/>

  <!-- STAGE 3 -->
  <rect x="20" y="396" width="820" height="130" rx="10" fill="#faf5ff" stroke="#c084fc" stroke-width="1.5"/>
  <text x="36" y="418" font-weight="700" font-size="13" fill="#6b21a8">Stage 3 — Material Generation</text>
  <text x="36" y="434" font-size="10" fill="#9333ea">queue: ai · parallel tasks · match_score >= AUTO_APPLY_MATCH_THRESHOLD</text>
  <rect x="36" y="444" width="380" height="68" rx="7" fill="#f3e8ff" stroke="#c084fc" stroke-width="1"/>
  <text x="226" y="466" text-anchor="middle" font-weight="600" font-size="11" fill="#6b21a8">Resume Engine (GPT-4o)</text>
  <text x="226" y="483" text-anchor="middle" font-size="10" fill="#581c87">ATS keyword injection · PDF render (WeasyPrint)</text>
  <text x="226" y="499" text-anchor="middle" font-size="10" fill="#581c87">ATS score estimate · version tracking</text>
  <rect x="434" y="444" width="386" height="68" rx="7" fill="#f3e8ff" stroke="#c084fc" stroke-width="1"/>
  <text x="627" y="466" text-anchor="middle" font-weight="600" font-size="11" fill="#6b21a8">Cover Letter Generator (GPT-4o)</text>
  <text x="627" y="483" text-anchor="middle" font-size="10" fill="#581c87">Company context · role alignment</text>
  <text x="627" y="499" text-anchor="middle" font-size="10" fill="#581c87">Configurable tone · version-tracked</text>
  <line x1="430" y1="526" x2="430" y2="548" stroke="#64748b" stroke-width="1.5" marker-end="url(#ap)"/>

  <!-- STAGE 4 -->
  <rect x="20" y="550" width="820" height="152" rx="10" fill="#fff1f2" stroke="#f87171" stroke-width="1.5"/>
  <text x="36" y="572" font-weight="700" font-size="13" fill="#991b1b">Stage 4 — Auto-Apply Bot</text>
  <text x="36" y="588" font-size="10" fill="#ef4444">queue: automation · Playwright Chromium · ATS detection from URL</text>
  <rect x="36" y="598" width="240" height="86" rx="7" fill="#fee2e2" stroke="#f87171" stroke-width="1"/>
  <text x="156" y="620" text-anchor="middle" font-weight="600" font-size="11" fill="#991b1b">Human-in-the-Loop Gate</text>
  <text x="156" y="638" text-anchor="middle" font-size="10" fill="#7f1d1d">Telegram: job + match score</text>
  <text x="156" y="654" text-anchor="middle" font-size="10" fill="#7f1d1d">Inline Approve / Skip buttons</text>
  <text x="156" y="670" text-anchor="middle" font-size="10" fill="#7f1d1d">PENDING_APPROVAL → QUEUED</text>
  <rect x="292" y="598" width="528" height="86" rx="7" fill="#fee2e2" stroke="#f87171" stroke-width="1"/>
  <text x="556" y="620" text-anchor="middle" font-weight="600" font-size="11" fill="#991b1b">Playwright Form Fill + Submit</text>
  <text x="556" y="638" text-anchor="middle" font-size="10" fill="#7f1d1d">ATS: Workday · Greenhouse · Lever · LinkedIn Easy Apply · Indeed · Internshala</text>
  <text x="556" y="654" text-anchor="middle" font-size="10" fill="#7f1d1d">Personal info · resume PDF upload · cover letter · standard questions</text>
  <text x="556" y="670" text-anchor="middle" font-size="10" fill="#7f1d1d">CAPTCHA detected → pause → Telegram escalation → await resolution</text>
  <line x1="276" y1="641" x2="290" y2="641" stroke="#64748b" stroke-width="1.5" marker-end="url(#ap)"/>
  <line x1="430" y1="702" x2="430" y2="724" stroke="#64748b" stroke-width="1.5" marker-end="url(#ap)"/>

  <!-- STAGE 5 -->
  <rect x="20" y="726" width="820" height="112" rx="10" fill="#f0fdf4" stroke="#34d399" stroke-width="1.5"/>
  <text x="36" y="748" font-weight="700" font-size="13" fill="#065f46">Stage 5 — Tracking &amp; Notifications</text>
  <text x="36" y="764" font-size="10" fill="#059669">queue: notifications · every state transition is audited</text>
  <rect x="36" y="774" width="228" height="48" rx="7" fill="#d1fae5" stroke="#34d399" stroke-width="1"/>
  <text x="150" y="796" text-anchor="middle" font-weight="600" font-size="11" fill="#065f46">Update application status</text>
  <text x="150" y="812" text-anchor="middle" font-size="10" fill="#065f46">PostgreSQL write</text>
  <rect x="280" y="774" width="258" height="48" rx="7" fill="#d1fae5" stroke="#34d399" stroke-width="1"/>
  <text x="409" y="796" text-anchor="middle" font-weight="600" font-size="11" fill="#065f46">ApplicationEvent audit log</text>
  <text x="409" y="812" text-anchor="middle" font-size="10" fill="#065f46">timestamp · source · metadata</text>
  <rect x="554" y="774" width="270" height="48" rx="7" fill="#d1fae5" stroke="#34d399" stroke-width="1"/>
  <text x="689" y="796" text-anchor="middle" font-weight="600" font-size="11" fill="#065f46">Telegram + email digest</text>
  <text x="689" y="812" text-anchor="middle" font-size="10" fill="#065f46">Cycle results · next actions</text>
</svg>

</div>

### Stage 1 — Job Discovery (`queue: scraping`)

Four platform scrapers run concurrently. Each applies polite rate-limiting delays (configurable 2–6 s between requests) and `fake-useragent` rotation. New jobs are deduplicated by `source_job_id` before being written to PostgreSQL with `status = NEW`.

| Scraper | Platform | Authentication |
|---|---|---|
| `linkedin.py` | LinkedIn | Session cookies / credentials |
| `indeed.py` | Indeed | Optional credentials |
| `internshala.py` | Internshala | Optional credentials |
| `wellfound.py` | Wellfound (AngelList) | Session cookies |

### Stage 2 — AI Analysis & Scoring (`queue: ai`)

A two-pass LLM pipeline balances cost against analytical depth.

**Pass 1 — Batch filter (GPT-4o-mini, ~$0.001/job).** Each new job is scored against `USER_DESIRED_ROLES` and `USER_EXPERIENCE_LEVEL`. Jobs below the threshold are immediately marked `SKIPPED` and exit the pipeline, keeping downstream API costs low.

**Pass 2 — Deep analysis (GPT-4o, structured JSON output).** Qualifying jobs are analysed in full. The model extracts required and preferred skills, tech stack, ATS keywords, seniority level, salary estimates, and job difficulty, then computes three derived scores:

- `match_score` (0–100) — skills overlap + experience fit + role category alignment
- `priority_score` — match score adjusted for recency, company prestige, and estimated competition
- `skill_gaps` — required skills the user does not currently hold

Results are persisted as a `JobAnalysis` record and the job advances to `status = ANALYZED`.

### Stage 3 — Material Generation (`queue: ai`)

For every job with `match_score >= AUTO_APPLY_MATCH_THRESHOLD`, two tasks run in parallel.

**Resume engine.** GPT-4o rewrites the professional summary, experience bullets, and project descriptions with ATS keyword injection, estimates an ATS score, and renders the final document to PDF via WeasyPrint or ReportLab.

**Cover letter generator.** GPT-4o produces a company-specific cover letter with opening, skills-alignment body, and a targeted closing paragraph — configurable tone (professional, tech, or casual). Both artefacts are version-tracked and attached to the pending application record.

### Stage 4 — Auto-Apply (`queue: automation`)

The Playwright-based `ApplyBot` detects the ATS platform from the job URL and executes a platform-specific form-fill routine. Supported ATS platforms: Workday, Greenhouse, Lever, LinkedIn Easy Apply, Indeed, and Internshala.

Before any submission, if `AUTO_APPLY_REQUIRE_APPROVAL = true`, the bot sends a Telegram notification with the job title, company, match score, and inline Approve / Skip buttons. The application stays in `PENDING_APPROVAL` until the user responds. If a CAPTCHA is detected mid-apply, the bot pauses and sends a Telegram escalation.

### Stage 5 — Tracking & Notifications (`queue: notifications`)

Application status is updated in PostgreSQL, every state transition is recorded as an `ApplicationEvent`, and a Telegram + email digest summarises the cycle results.

---

## Application Lifecycle

Every application passes through a well-defined finite state machine. Every transition is recorded as an `ApplicationEvent` with a timestamp, trigger source (`agent` or `user`), and optional metadata.

<div align="center">

<svg width="860" viewBox="0 0 860 720" xmlns="http://www.w3.org/2000/svg" font-family="ui-monospace,SFMono-Regular,Menlo,monospace" font-size="12">
  <defs>
    <marker id="fa" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#64748b" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
    <marker id="fr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#ef4444" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
    <marker id="fg" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#22c55e" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  <!-- PENDING_APPROVAL -->
  <rect x="270" y="20" width="220" height="52" rx="8" fill="#fef9c3" stroke="#eab308" stroke-width="1.5"/>
  <text x="380" y="43" text-anchor="middle" font-weight="700" font-size="12" fill="#713f12">PENDING_APPROVAL</text>
  <text x="380" y="61" text-anchor="middle" font-size="10" fill="#92400e">Awaiting Telegram confirmation</text>

  <!-- PENDING → SKIPPED -->
  <line x1="490" y1="46" x2="634" y2="46" stroke="#ef4444" stroke-width="1.5" marker-end="url(#fr)"/>
  <text x="560" y="40" text-anchor="middle" font-size="10" fill="#ef4444">user skips</text>
  <rect x="636" y="20" width="100" height="52" rx="8" fill="#fee2e2" stroke="#f87171" stroke-width="1.5"/>
  <text x="686" y="50" text-anchor="middle" font-weight="700" font-size="12" fill="#991b1b">SKIPPED</text>

  <!-- PENDING → QUEUED -->
  <line x1="380" y1="72" x2="380" y2="104" stroke="#64748b" stroke-width="1.5" marker-end="url(#fa)"/>
  <text x="390" y="93" font-size="10" fill="#64748b">user approves</text>

  <!-- QUEUED -->
  <rect x="285" y="106" width="190" height="44" rx="8" fill="#dbeafe" stroke="#60a5fa" stroke-width="1.5"/>
  <text x="380" y="133" text-anchor="middle" font-weight="700" font-size="12" fill="#1e40af">QUEUED</text>
  <line x1="380" y1="150" x2="380" y2="180" stroke="#64748b" stroke-width="1.5" marker-end="url(#fa)"/>
  <text x="390" y="169" font-size="10" fill="#64748b">bot picks up</text>

  <!-- APPLYING -->
  <rect x="285" y="182" width="190" height="44" rx="8" fill="#dbeafe" stroke="#60a5fa" stroke-width="1.5"/>
  <text x="380" y="209" text-anchor="middle" font-weight="700" font-size="12" fill="#1e40af">APPLYING</text>

  <!-- APPLYING → FAILED -->
  <line x1="475" y1="204" x2="614" y2="204" stroke="#ef4444" stroke-width="1.5" marker-end="url(#fr)"/>
  <text x="543" y="198" text-anchor="middle" font-size="10" fill="#ef4444">error / CAPTCHA</text>
  <rect x="616" y="182" width="120" height="44" rx="8" fill="#fee2e2" stroke="#f87171" stroke-width="1.5"/>
  <text x="676" y="203" text-anchor="middle" font-weight="700" font-size="12" fill="#991b1b">FAILED</text>
  <text x="676" y="219" text-anchor="middle" font-size="10" fill="#991b1b">retry ×3, exp backoff</text>

  <!-- APPLYING → APPLIED -->
  <line x1="380" y1="226" x2="380" y2="256" stroke="#22c55e" stroke-width="1.5" marker-end="url(#fg)"/>
  <text x="390" y="246" font-size="10" fill="#22c55e">submitted successfully</text>

  <!-- APPLIED -->
  <rect x="285" y="258" width="190" height="44" rx="8" fill="#dcfce7" stroke="#4ade80" stroke-width="1.5"/>
  <text x="380" y="285" text-anchor="middle" font-weight="700" font-size="12" fill="#15803d">APPLIED</text>
  <line x1="380" y1="302" x2="380" y2="332" stroke="#64748b" stroke-width="1.5" marker-end="url(#fa)"/>
  <text x="390" y="321" font-size="10" fill="#64748b">recruiter opens</text>

  <!-- VIEWED -->
  <rect x="285" y="334" width="190" height="44" rx="8" fill="#dcfce7" stroke="#4ade80" stroke-width="1.5"/>
  <text x="380" y="361" text-anchor="middle" font-weight="700" font-size="12" fill="#15803d">VIEWED</text>

  <!-- VIEWED → REJECTED -->
  <line x1="475" y1="356" x2="614" y2="356" stroke="#ef4444" stroke-width="1.5" marker-end="url(#fr)"/>
  <text x="543" y="350" text-anchor="middle" font-size="10" fill="#ef4444">not progressed</text>
  <rect x="616" y="334" width="120" height="44" rx="8" fill="#fee2e2" stroke="#f87171" stroke-width="1.5"/>
  <text x="676" y="361" text-anchor="middle" font-weight="700" font-size="12" fill="#991b1b">REJECTED</text>

  <line x1="380" y1="378" x2="380" y2="408" stroke="#64748b" stroke-width="1.5" marker-end="url(#fa)"/>
  <text x="390" y="397" font-size="10" fill="#64748b">shortlisted</text>

  <!-- SHORTLISTED -->
  <rect x="275" y="410" width="210" height="44" rx="8" fill="#faf5ff" stroke="#c084fc" stroke-width="1.5"/>
  <text x="380" y="437" text-anchor="middle" font-weight="700" font-size="12" fill="#6b21a8">SHORTLISTED</text>
  <line x1="380" y1="454" x2="380" y2="484" stroke="#64748b" stroke-width="1.5" marker-end="url(#fa)"/>

  <!-- INTERVIEW_SCHEDULED -->
  <rect x="240" y="486" width="280" height="44" rx="8" fill="#faf5ff" stroke="#c084fc" stroke-width="1.5"/>
  <text x="380" y="513" text-anchor="middle" font-weight="700" font-size="12" fill="#6b21a8">INTERVIEW_SCHEDULED</text>
  <line x1="380" y1="530" x2="380" y2="560" stroke="#64748b" stroke-width="1.5" marker-end="url(#fa)"/>

  <!-- INTERVIEW_COMPLETED -->
  <rect x="240" y="562" width="280" height="44" rx="8" fill="#faf5ff" stroke="#c084fc" stroke-width="1.5"/>
  <text x="380" y="589" text-anchor="middle" font-weight="700" font-size="12" fill="#6b21a8">INTERVIEW_COMPLETED</text>

  <!-- → OFFER_RECEIVED -->
  <line x1="310" y1="606" x2="200" y2="636" stroke="#22c55e" stroke-width="1.5" marker-end="url(#fg)"/>
  <rect x="60" y="614" width="136" height="44" rx="8" fill="#dcfce7" stroke="#4ade80" stroke-width="1.5"/>
  <text x="128" y="641" text-anchor="middle" font-weight="700" font-size="12" fill="#15803d">OFFER_RECEIVED</text>

  <!-- → OFFER_ACCEPTED / DECLINED -->
  <line x1="96" y1="658" x2="60" y2="686" stroke="#22c55e" stroke-width="1.5" marker-end="url(#fg)"/>
  <rect x="20" y="664" width="120" height="40" rx="8" fill="#dcfce7" stroke="#4ade80" stroke-width="1.5"/>
  <text x="80" y="689" text-anchor="middle" font-weight="700" font-size="11" fill="#15803d">OFFER_ACCEPTED</text>
  <line x1="162" y1="658" x2="200" y2="686" stroke="#ef4444" stroke-width="1.5" marker-end="url(#fr)"/>
  <rect x="160" y="664" width="120" height="40" rx="8" fill="#fee2e2" stroke="#f87171" stroke-width="1.5"/>
  <text x="220" y="689" text-anchor="middle" font-weight="700" font-size="11" fill="#991b1b">OFFER_DECLINED</text>

  <!-- → REJECTED post-interview -->
  <line x1="450" y1="606" x2="560" y2="636" stroke="#ef4444" stroke-width="1.5" marker-end="url(#fr)"/>
  <rect x="562" y="614" width="120" height="44" rx="8" fill="#fee2e2" stroke="#f87171" stroke-width="1.5"/>
  <text x="622" y="641" text-anchor="middle" font-weight="700" font-size="12" fill="#991b1b">REJECTED</text>

  <!-- WITHDRAWN -->
  <rect x="720" y="410" width="120" height="44" rx="8" fill="#f1f5f9" stroke="#94a3b8" stroke-width="1.5" stroke-dasharray="5,3"/>
  <text x="780" y="430" text-anchor="middle" font-weight="700" font-size="11" fill="#475569">WITHDRAWN</text>
  <text x="780" y="446" text-anchor="middle" font-size="10" fill="#64748b">any state</text>

  <!-- Legend box -->
  <rect x="20" y="20" width="220" height="80" rx="8" fill="none" stroke="#cbd5e1" stroke-width="1" stroke-dasharray="4,3"/>
  <text x="30" y="40" font-weight="600" font-size="11" fill="#475569">ApplicationMethod</text>
  <text x="30" y="58" font-size="10" fill="#64748b">AUTO_BOT · EASY_APPLY</text>
  <text x="30" y="74" font-size="10" fill="#64748b">MANUAL · EMAIL</text>
  <text x="30" y="90" font-size="10" fill="#64748b">every transition → ApplicationEvent</text>
</svg>

</div>

`ApplicationMethod` values: `AUTO_BOT` | `EASY_APPLY` | `MANUAL` | `EMAIL`

---

## Database Schema

All entities and their relationships are defined in `app/models/`. The ERD below covers all key tables.

<div align="center">

<svg width="860" viewBox="0 0 860 900" xmlns="http://www.w3.org/2000/svg" font-family="ui-monospace,SFMono-Regular,Menlo,monospace" font-size="11">
  <defs>
    <marker id="erd" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#94a3b8" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  <!-- users -->
  <rect x="20" y="20" width="190" height="152" rx="8" fill="#eff6ff" stroke="#60a5fa" stroke-width="1.5"/>
  <rect x="20" y="20" width="190" height="28" rx="8" fill="#60a5fa"/>
  <rect x="20" y="36" width="190" height="12" fill="#60a5fa"/>
  <text x="115" y="38" text-anchor="middle" font-weight="700" font-size="12" fill="#fff">users</text>
  <text x="30" y="66" font-size="10" fill="#1e40af">PK  id            uuid</text>
  <text x="30" y="82" font-size="10" fill="#1e40af">    email         varchar</text>
  <text x="30" y="98" font-size="10" fill="#1e40af">    hashed_pass   varchar</text>
  <text x="30" y="114" font-size="10" fill="#1e40af">    is_active      bool</text>
  <text x="30" y="130" font-size="10" fill="#1e40af">    full_name      varchar</text>
  <text x="30" y="146" font-size="10" fill="#1e40af">    created_at     timestamp</text>
  <text x="30" y="162" font-size="10" fill="#1e40af">    updated_at     timestamp</text>

  <!-- user_profiles -->
  <rect x="20" y="192" width="190" height="138" rx="8" fill="#eff6ff" stroke="#60a5fa" stroke-width="1.5"/>
  <rect x="20" y="192" width="190" height="28" rx="8" fill="#60a5fa"/>
  <rect x="20" y="208" width="190" height="12" fill="#60a5fa"/>
  <text x="115" y="210" text-anchor="middle" font-weight="700" font-size="12" fill="#fff">user_profiles</text>
  <text x="30" y="238" font-size="10" fill="#1e40af">PK  id               uuid</text>
  <text x="30" y="254" font-size="10" fill="#1e40af">FK  user_id          uuid</text>
  <text x="30" y="270" font-size="10" fill="#1e40af">    experience_level varchar</text>
  <text x="30" y="286" font-size="10" fill="#1e40af">    open_to_remote   bool</text>
  <text x="30" y="302" font-size="10" fill="#1e40af">    min_salary       int</text>
  <text x="30" y="318" font-size="10" fill="#1e40af">    desired_roles    json</text>

  <!-- user_skills -->
  <rect x="20" y="350" width="190" height="124" rx="8" fill="#eff6ff" stroke="#60a5fa" stroke-width="1.5"/>
  <rect x="20" y="350" width="190" height="28" rx="8" fill="#60a5fa"/>
  <rect x="20" y="366" width="190" height="12" fill="#60a5fa"/>
  <text x="115" y="368" text-anchor="middle" font-weight="700" font-size="12" fill="#fff">user_skills</text>
  <text x="30" y="396" font-size="10" fill="#1e40af">PK  id           uuid</text>
  <text x="30" y="412" font-size="10" fill="#1e40af">FK  user_id      uuid</text>
  <text x="30" y="428" font-size="10" fill="#1e40af">    skill_name   varchar</text>
  <text x="30" y="444" font-size="10" fill="#1e40af">    proficiency  varchar</text>
  <text x="30" y="460" font-size="10" fill="#1e40af">    is_verified  bool</text>

  <!-- jobs -->
  <rect x="270" y="20" width="200" height="194" rx="8" fill="#ecfdf5" stroke="#4ade80" stroke-width="1.5"/>
  <rect x="270" y="20" width="200" height="28" rx="8" fill="#4ade80"/>
  <rect x="270" y="36" width="200" height="12" fill="#4ade80"/>
  <text x="370" y="38" text-anchor="middle" font-weight="700" font-size="12" fill="#fff">jobs</text>
  <text x="280" y="66" font-size="10" fill="#065f46">PK  id             uuid</text>
  <text x="280" y="82" font-size="10" fill="#065f46">    source         varchar</text>
  <text x="280" y="98" font-size="10" fill="#065f46">    source_job_id  varchar</text>
  <text x="280" y="114" font-size="10" fill="#065f46">    title          varchar</text>
  <text x="280" y="130" font-size="10" fill="#065f46">    company_name   varchar</text>
  <text x="280" y="146" font-size="10" fill="#065f46">    job_type       varchar</text>
  <text x="280" y="162" font-size="10" fill="#065f46">    work_mode      varchar</text>
  <text x="280" y="178" font-size="10" fill="#065f46">    status         varchar</text>
  <text x="280" y="194" font-size="10" fill="#065f46">    posted_at      timestamp</text>

  <!-- job_analyses -->
  <rect x="270" y="234" width="200" height="194" rx="8" fill="#ecfdf5" stroke="#4ade80" stroke-width="1.5"/>
  <rect x="270" y="234" width="200" height="28" rx="8" fill="#4ade80"/>
  <rect x="270" y="250" width="200" height="12" fill="#4ade80"/>
  <text x="370" y="252" text-anchor="middle" font-weight="700" font-size="12" fill="#fff">job_analyses</text>
  <text x="280" y="280" font-size="10" fill="#065f46">PK  id                  uuid</text>
  <text x="280" y="296" font-size="10" fill="#065f46">FK  job_id              uuid</text>
  <text x="280" y="312" font-size="10" fill="#065f46">    match_score         float</text>
  <text x="280" y="328" font-size="10" fill="#065f46">    priority_score      float</text>
  <text x="280" y="344" font-size="10" fill="#065f46">    skill_gaps          json</text>
  <text x="280" y="360" font-size="10" fill="#065f46">    ats_keywords        json</text>
  <text x="280" y="376" font-size="10" fill="#065f46">    seniority_detected  varchar</text>
  <text x="280" y="392" font-size="10" fill="#065f46">    ai_recommendation   varchar</text>
  <text x="280" y="408" font-size="10" fill="#065f46">    job_difficulty      varchar</text>

  <!-- resumes -->
  <rect x="530" y="20" width="200" height="166" rx="8" fill="#faf5ff" stroke="#c084fc" stroke-width="1.5"/>
  <rect x="530" y="20" width="200" height="28" rx="8" fill="#c084fc"/>
  <rect x="530" y="36" width="200" height="12" fill="#c084fc"/>
  <text x="630" y="38" text-anchor="middle" font-weight="700" font-size="12" fill="#fff">resumes</text>
  <text x="540" y="66" font-size="10" fill="#581c87">PK  id               uuid</text>
  <text x="540" y="82" font-size="10" fill="#581c87">FK  user_id          uuid</text>
  <text x="540" y="98" font-size="10" fill="#581c87">    resume_type      varchar</text>
  <text x="540" y="114" font-size="10" fill="#581c87">    target_role      varchar</text>
  <text x="540" y="130" font-size="10" fill="#581c87">    ats_score        float</text>
  <text x="540" y="146" font-size="10" fill="#581c87">    pdf_path         varchar</text>
  <text x="540" y="162" font-size="10" fill="#581c87">    is_base          bool</text>

  <!-- applications -->
  <rect x="530" y="206" width="200" height="180" rx="8" fill="#fff7ed" stroke="#fb923c" stroke-width="1.5"/>
  <rect x="530" y="206" width="200" height="28" rx="8" fill="#fb923c"/>
  <rect x="530" y="222" width="200" height="12" fill="#fb923c"/>
  <text x="630" y="224" text-anchor="middle" font-weight="700" font-size="12" fill="#fff">applications</text>
  <text x="540" y="252" font-size="10" fill="#7c2d12">PK  id               uuid</text>
  <text x="540" y="268" font-size="10" fill="#7c2d12">FK  job_id           uuid</text>
  <text x="540" y="284" font-size="10" fill="#7c2d12">FK  resume_id        uuid</text>
  <text x="540" y="300" font-size="10" fill="#7c2d12">FK  cover_letter_id  uuid</text>
  <text x="540" y="316" font-size="10" fill="#7c2d12">    status           varchar</text>
  <text x="540" y="332" font-size="10" fill="#7c2d12">    method           varchar</text>
  <text x="540" y="348" font-size="10" fill="#7c2d12">    applied_at       timestamp</text>
  <text x="540" y="364" font-size="10" fill="#7c2d12">    is_starred       bool</text>
  <text x="540" y="380" font-size="10" fill="#7c2d12">    follow_up_sent   bool</text>

  <!-- application_events -->
  <rect x="530" y="406" width="200" height="152" rx="8" fill="#fff1f2" stroke="#f87171" stroke-width="1.5"/>
  <rect x="530" y="406" width="200" height="28" rx="8" fill="#f87171"/>
  <rect x="530" y="422" width="200" height="12" fill="#f87171"/>
  <text x="630" y="424" text-anchor="middle" font-weight="700" font-size="12" fill="#fff">application_events</text>
  <text x="540" y="452" font-size="10" fill="#7f1d1d">PK  id                uuid</text>
  <text x="540" y="468" font-size="10" fill="#7f1d1d">FK  application_id    uuid</text>
  <text x="540" y="484" font-size="10" fill="#7f1d1d">    from_status       varchar</text>
  <text x="540" y="500" font-size="10" fill="#7f1d1d">    to_status         varchar</text>
  <text x="540" y="516" font-size="10" fill="#7f1d1d">    triggered_by      varchar</text>
  <text x="540" y="532" font-size="10" fill="#7f1d1d">    event_type        varchar</text>
  <text x="540" y="548" font-size="10" fill="#7f1d1d">    created_at        timestamp</text>

  <!-- interviews -->
  <rect x="270" y="448" width="200" height="152" rx="8" fill="#fef9c3" stroke="#facc15" stroke-width="1.5"/>
  <rect x="270" y="448" width="200" height="28" rx="8" fill="#facc15"/>
  <rect x="270" y="464" width="200" height="12" fill="#facc15"/>
  <text x="370" y="466" text-anchor="middle" font-weight="700" font-size="12" fill="#713f12">interviews</text>
  <text x="280" y="494" font-size="10" fill="#713f12">PK  id               uuid</text>
  <text x="280" y="510" font-size="10" fill="#713f12">FK  application_id   uuid</text>
  <text x="280" y="526" font-size="10" fill="#713f12">    interview_type   varchar</text>
  <text x="280" y="542" font-size="10" fill="#713f12">    scheduled_at     timestamp</text>
  <text x="280" y="558" font-size="10" fill="#713f12">    company_report   json</text>
  <text x="280" y="574" font-size="10" fill="#713f12">    tech_questions   json</text>
  <text x="280" y="590" font-size="10" fill="#713f12">    outcome          varchar</text>

  <!-- mock_interview_sessions -->
  <rect x="270" y="620" width="200" height="152" rx="8" fill="#fef9c3" stroke="#facc15" stroke-width="1.5"/>
  <rect x="270" y="620" width="200" height="28" rx="8" fill="#facc15"/>
  <rect x="270" y="636" width="200" height="12" fill="#facc15"/>
  <text x="370" y="638" text-anchor="middle" font-weight="700" font-size="12" fill="#713f12">mock_interview_sessions</text>
  <text x="280" y="666" font-size="10" fill="#713f12">PK  id                uuid</text>
  <text x="280" y="682" font-size="10" fill="#713f12">FK  interview_id      uuid</text>
  <text x="280" y="698" font-size="10" fill="#713f12">    overall_score     float</text>
  <text x="280" y="714" font-size="10" fill="#713f12">    technical_depth   float</text>
  <text x="280" y="730" font-size="10" fill="#713f12">    comm_score        float</text>
  <text x="280" y="746" font-size="10" fill="#713f12">    confidence        float</text>
  <text x="280" y="762" font-size="10" fill="#713f12">    improvements      json</text>

  <!-- Relationship lines -->
  <!-- users → user_profiles (1:1) -->
  <line x1="115" y1="172" x2="115" y2="192" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3" marker-end="url(#erd)"/>
  <text x="120" y="186" font-size="9" fill="#94a3b8">1:1</text>

  <!-- users → user_skills (1:N) -->
  <line x1="115" y1="330" x2="115" y2="350" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3" marker-end="url(#erd)"/>
  <text x="120" y="344" font-size="9" fill="#94a3b8">1:N</text>

  <!-- users → resumes (1:N) -->
  <line x1="210" y1="100" x2="530" y2="100" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3" marker-end="url(#erd)"/>
  <text x="358" y="95" font-size="9" fill="#94a3b8">1:N</text>

  <!-- jobs → job_analyses (1:1) -->
  <line x1="370" y1="214" x2="370" y2="234" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3" marker-end="url(#erd)"/>
  <text x="376" y="228" font-size="9" fill="#94a3b8">1:1</text>

  <!-- jobs → applications (1:N) -->
  <line x1="470" y1="117" x2="530" y2="296" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3" marker-end="url(#erd)"/>
  <text x="500" y="190" font-size="9" fill="#94a3b8">1:N</text>

  <!-- resumes → applications (1:N) -->
  <line x1="630" y1="186" x2="630" y2="206" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3" marker-end="url(#erd)"/>
  <text x="636" y="200" font-size="9" fill="#94a3b8">1:N</text>

  <!-- applications → application_events (1:N) -->
  <line x1="630" y1="386" x2="630" y2="406" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3" marker-end="url(#erd)"/>
  <text x="636" y="400" font-size="9" fill="#94a3b8">1:N</text>

  <!-- applications → interviews (1:N) -->
  <line x1="530" y1="340" x2="470" y2="524" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3" marker-end="url(#erd)"/>
  <text x="490" y="430" font-size="9" fill="#94a3b8">1:N</text>

  <!-- interviews → mock_interview_sessions (1:N) -->
  <line x1="370" y1="600" x2="370" y2="620" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3" marker-end="url(#erd)"/>
  <text x="376" y="614" font-size="9" fill="#94a3b8">1:N</text>
</svg>

</div>

### Job taxonomy

```python
JobSource:    linkedin | indeed | internshala | wellfound | glassdoor | company_site | manual
JobType:      full_time | part_time | internship | contract | freelance
WorkMode:     remote | onsite | hybrid | unknown
JobStatus:    new | analyzed | queued | applied | skipped | expired
Seniority:    entry | mid | senior | lead | unknown
RoleCategory: computer_vision | nlp | mlops | data_science | software_engineering | other
Difficulty:   easy | medium | hard
```

---

## Core Modules

| # | Module | Source File | Description |
|---|--------|-------------|-------------|
| 1 | Job Discovery | `agents/scrapers/` | Multi-platform scrapers with polite rate limiting, `fake-useragent` rotation, and deduplication by `source_job_id` |
| 2 | Job Analyzer | `services/job_analyzer.py` | Dual-model LLM pipeline: GPT-4o-mini batch filter + GPT-4o deep analysis with structured JSON output |
| 3 | Resume Engine | `services/resume_service.py` | ATS-optimised resume tailoring, keyword injection, ATS score estimation, WeasyPrint/ReportLab PDF rendering, version tracking |
| 4 | Cover Letter Generator | `services/cover_letter_service.py` | GPT-4o per-job cover letter with company context loading, role alignment, and configurable tone |
| 5 | Auto Apply Bot | `agents/apply_bot.py` | Playwright Chromium automation: ATS detection (Workday, Greenhouse, Lever, LinkedIn Easy Apply, Indeed, Internshala), form fill, CAPTCHA escalation |
| 6 | Application Tracker | `models/application.py` | 14-state lifecycle FSM with full `ApplicationEvent` audit log and `ApplicationMethod` classification |
| 7 | Follow-up Automation | `services/follow_up_service.py` | Scheduled 7-day follow-up emails, interview confirmations, post-interview thank-you dispatch |
| 8 | Notification Service | `services/notification_service.py` | Telegram bot with inline keyboards and Jinja2-templated SMTP email digests |
| 9 | AI Chat Assistant | `services/ai_assistant.py` | Conversational interface with RAG context from user profile, application history, and job data |
| 10 | Market Intelligence | `services/market_service.py` | Periodic snapshots: top skill demand, emerging roles, salary band aggregation, hiring velocity |
| 11 | Interview Engine | `services/interview_service.py` | Company report generation, technical and behavioural Q&A bank, mock sessions, Whisper recording analysis |
| 12 | Background Agent | `agents/tasks.py` | Celery task definitions, Beat schedule, 4-queue routing with per-queue rate limits, retry policies, `AgentTask` logging |

---

## Quick Start

### Prerequisites

| Dependency | Minimum version | Purpose |
|---|---|---|
| Python | 3.11 | Runtime |
| PostgreSQL | 15 | Primary datastore |
| Redis | 7 | Celery broker and result backend |
| Node.js | 18 | Next.js frontend |
| Playwright / Chromium | 1.44 | Browser automation |

### 1. Clone the repository

```bash
git clone https://github.com/Sudharsanselvaraj/Distributed-Autonomous-Opportunity-Intelligence-and-Strategic-Application-Orchestration-System.git
cd Distributed-Autonomous-Opportunity-Intelligence-and-Strategic-Application-Orchestration-System
```

### 2. Python environment

```bash
python -m venv venv
source venv/bin/activate          # Linux / macOS
# venv\Scripts\activate           # Windows

pip install -r requirements.txt
playwright install chromium
```

### 3. Environment configuration

```bash
cp career_platform/.env.example career_platform/.env
# Edit .env — see the Configuration section for all variables
```

### 4. Database initialisation

```bash
psql -U postgres -c "CREATE DATABASE career_platform;"
cd career_platform
alembic upgrade head
```

### 5. Start services

**Windows (automated):**
```bat
setup.bat           # First-time: install deps, create directories, run migrations
start_platform.bat  # Start all services
```

**Linux / macOS — four separate terminals:**

```bash
# Terminal 1 — FastAPI server
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Terminal 2 — Celery workers (all four queues)
celery -A app.agents.tasks.celery_app worker --loglevel=info \
  -Q scraping,ai,automation,notifications --concurrency=4

# Terminal 3 — Celery Beat scheduler
celery -A app.agents.tasks.celery_app beat --loglevel=info

# Terminal 4 — Flower monitoring dashboard (optional)
celery -A app.agents.tasks.celery_app flower --port=5555
```

### 6. Register and authenticate

```bash
# Create an account
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "yourpassword", "full_name": "Your Name"}'

# Obtain a JWT token
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "yourpassword"}'
```

### 7. Service endpoints

| Service | URL | Notes |
|---|---|---|
| Swagger UI | `http://localhost:8000/api/docs` | Full interactive API reference |
| ReDoc | `http://localhost:8000/api/redoc` | Clean reference layout |
| Health check | `http://localhost:8000/health` | DB connectivity status |
| Celery Flower | `http://localhost:5555` | Task monitoring dashboard |

---

## Configuration

All configuration is managed via `career_platform/.env`. The full template is provided in `.env.example`.

### Application

```env
APP_NAME="AI Career Platform"
APP_ENV=development              # development | staging | production
APP_HOST=0.0.0.0
APP_PORT=8000
DEBUG=true
SECRET_KEY=your-secret-key-minimum-32-characters
```

### Database

```env
DATABASE_URL=postgresql+asyncpg://career_user:career_pass@localhost:5432/career_platform
DATABASE_URL_SYNC=postgresql://career_user:career_pass@localhost:5432/career_platform
```

### Task queue

```env
REDIS_URL=redis://localhost:6379/0
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/1
```

### AI layer

```env
OPENAI_API_KEY=sk-...
OPENAI_MODEL_HEAVY=gpt-4o           # Deep analysis, resume/cover letter gen, interview prep
OPENAI_MODEL_LIGHT=gpt-4o-mini      # Batch scoring, quick filtering, chat responses
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_MAX_TOKENS=4096
OPENAI_TEMPERATURE=0.3
```

### Vector store (ChromaDB)

```env
CHROMA_HOST=localhost
CHROMA_PORT=8001
CHROMA_COLLECTION_USER_PROFILE=user_profile
CHROMA_COLLECTION_JOBS=jobs
CHROMA_COLLECTION_RESUMES=resumes
```

### Notifications

```env
# Telegram — create a bot via @BotFather, obtain your chat ID via @userinfobot
TELEGRAM_BOT_TOKEN=your-telegram-bot-token
TELEGRAM_CHAT_ID=your-personal-chat-id

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-gmail-app-password   # Gmail App Password — not your account password
SMTP_FROM_EMAIL=your-email@gmail.com
SMTP_FROM_NAME="AI Career Agent"
```

### User profile

The profile is injected into every AI prompt. Accuracy here directly determines match quality and generation relevance.

```env
USER_NAME="Your Name"
USER_EMAIL=your-email@gmail.com
USER_PHONE=+91-XXXXXXXXXX
USER_LOCATION="Chennai, India"
USER_DESIRED_ROLES=["Computer Vision Engineer","ML Engineer","AI Research Intern","Data Scientist"]
USER_DESIRED_LOCATIONS=["Remote","Bangalore","Hyderabad","Chennai","Europe","USA"]
USER_EXPERIENCE_LEVEL=entry          # entry | mid | senior
USER_OPEN_TO_REMOTE=true
USER_MIN_SALARY=0                    # 0 = no minimum constraint
```

### Job scraping

```env
SCRAPE_INTERVAL_HOURS=6              # Master agent cycle interval
MAX_JOBS_PER_CYCLE=200               # Cap on new jobs stored per cycle
SCRAPE_DELAY_MIN_SECONDS=2.0         # Polite random delay between HTTP requests
SCRAPE_DELAY_MAX_SECONDS=6.0

# Platform credentials (required for authenticated scraping)
LINKEDIN_EMAIL=your-linkedin@email.com
LINKEDIN_PASSWORD=your-linkedin-password
```

### Auto-apply

> **Auto-apply is disabled by default.** Review each setting carefully before enabling.

```env
AUTO_APPLY_ENABLED=false
AUTO_APPLY_MATCH_THRESHOLD=75        # Only queue jobs with match_score >= this value
AUTO_APPLY_DAILY_LIMIT=10            # Hard ceiling on submissions per calendar day
AUTO_APPLY_REQUIRE_APPROVAL=true     # Human-in-the-loop: Telegram approval before each submission
```

### File storage

```env
STORAGE_BACKEND=local                # local | s3
LOCAL_STORAGE_PATH=./storage         # Resumes, cover letters, interview recordings

# Optional S3
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_BUCKET_NAME=career-platform-files
AWS_REGION=us-east-1
```

### Authentication

```env
JWT_SECRET_KEY=your-jwt-secret-minimum-32-characters
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=1440   # 24 hours
```

---

## API Reference

Full interactive documentation is available at `/api/docs` (Swagger UI) and `/api/redoc` (ReDoc) when the server is running. All endpoints require a JWT Bearer token in the `Authorization` header.

### Router overview

| Router | Prefix | Key endpoints |
|---|---|---|
| Auth | `/api/auth` | `POST /register`, `POST /login`, `GET /me` |
| Jobs | `/api/jobs` | `GET /` (paginated, filterable), `GET /{id}`, `POST /` (manual), `POST /{id}/analyze`, `POST /{id}/skip` |
| Applications | `/api/applications` | `GET /`, `GET /{id}`, `POST /`, `PATCH /{id}/status`, `GET /stats` |
| Resumes | `/api/resumes` | `GET /`, `POST /upload`, `POST /generate`, `GET /{id}/download`, `DELETE /{id}` |
| Cover Letters | `/api/cover-letters` | `GET /`, `POST /generate`, `GET /{id}`, `DELETE /{id}` |
| Agent | `/api/agent` | `POST /run`, `GET /tasks`, `GET /tasks/{id}`, `POST /cycle/trigger` |
| Analytics | `/api/analytics` | `GET /dashboard`, `GET /market`, `GET /resume-performance`, `GET /skill-gaps` |
| Chat | `/api/chat` | `POST /message`, `GET /history`, `DELETE /history` |
| Profile | `/api/profile` | `GET /`, `PATCH /`, `/skills` CRUD |
| Security | `/api/security` | `/credentials`, `/consents`, `/data/export`, `/data/delete`, `/audit` |
| Onboarding | `/api/onboarding` | `GET /status`, `POST /complete`, `POST /profile` |

### Job filtering parameters (`GET /api/jobs`)

| Parameter | Type | Description |
|---|---|---|
| `source` | string | `linkedin` \| `indeed` \| `internshala` \| `wellfound` \| `manual` |
| `job_type` | string | `full_time` \| `internship` \| `contract` \| `part_time` |
| `work_mode` | string | `remote` \| `onsite` \| `hybrid` |
| `min_match_score` | float | Minimum match score (0–100) |
| `keyword` | string | Full-text search across title, company, and description |
| `status` | string | `new` \| `analyzed` \| `queued` \| `applied` \| `skipped` |
| `sort_by` | string | `match_score` \| `priority_score` \| `posted_at` \| `created_at` |
| `page` | int | Default: 1 |
| `page_size` | int | Default: 20, max: 100 |

### Usage examples

**Trigger a manual scrape cycle:**
```bash
curl -X POST http://localhost:8000/api/agent/run \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"task": "scrape_jobs", "params": {"platforms": ["linkedin", "indeed"]}}'
```

**List top-matched remote jobs:**
```bash
curl "http://localhost:8000/api/jobs?min_match_score=80&work_mode=remote&sort_by=match_score" \
  -H "Authorization: Bearer <token>"
```

**Generate a tailored resume for a specific job:**
```bash
curl -X POST http://localhost:8000/api/resumes/generate \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"job_id": "<uuid>", "base_resume_id": "<uuid>"}'
```

**Send a natural language command to the AI assistant:**
```bash
curl -X POST http://localhost:8000/api/chat/message \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"message": "Find AI internships in Europe and apply to the top 3 matches"}'
```

**Fetch dashboard analytics:**
```bash
curl http://localhost:8000/api/analytics/dashboard \
  -H "Authorization: Bearer <token>"
```

---

## Automation Schedule

All periodic tasks are registered in Celery Beat via `beat_schedule` in `agents/tasks.py`. No external cron configuration is required.

| Task | Interval | Queue | Description |
|---|---|---|---|
| `run_main_agent_cycle` | Every 6 hours | scraping / ai / automation | Full pipeline: discover → analyse → generate → apply → notify |
| `check_follow_ups` | Every 1 hour | notifications | Dispatch follow-up emails; automatic 7-day recruiter follow-up |
| `take_market_snapshot` | Daily | ai | Aggregate skill demand, salary trends, and hiring velocity |
| `update_resume_performance` | Every 6 hours | ai | Recalculate response rates per resume version from application outcomes |

### Queue architecture and rate limits

| Task pattern | Queue | Rate limit |
|---|---|---|
| `scrape_linkedin_task` | scraping | 10 / min |
| `scrape_indeed_task` | scraping | 20 / min |
| `scrape_internshala_task` / `scrape_wellfound_task` | scraping | default |
| `analyze_job_task` / `generate_resume_task` / `generate_cover_letter_task` | ai | default |
| `auto_apply_task` | automation | 5 / min |
| `send_telegram_notification` / `send_email_notification` / `check_follow_ups` | notifications | default |

**Retry policy (all tasks):** `max_retries = 3`, exponential backoff, `task_acks_late = true`, `task_reject_on_worker_lost = true`, `result_expires = 86400 s`.

---

## Interview Preparation Engine

When an application advances to `INTERVIEW_SCHEDULED`, the system automatically generates a full preparation package via `interview_service.py`.

**Company intelligence report** (GPT-4o + web search) — products and services, recent news, known tech stack, culture signals, Glassdoor rating, estimated interview difficulty, and known interview format.

**Question bank** (GPT-4o, based on role and JD) — technical questions with expected answers and difficulty ratings, behavioural questions with STAR-framework hints, and a ranked study-topics list.

**Mock interview sessions** (AI interviewer — GPT-4o) — full transcript with timestamped `{role, content}` turns, and a scored debrief covering `overall_score`, `technical_depth_score`, `communication_score`, `confidence_score`, and `improvement_suggestions[]`.

**Recording analysis** (OpenAI Whisper + GPT-4o) — transcription of audio recordings with analysis of filler word count, speech clarity, confidence level, technical depth, and pacing.

`InterviewType` values: `phone_screen` | `technical` | `behavioral` | `system_design` | `hr` | `final` | `take_home`

---

## Project Structure

```
career_platform/
├── app/
│   ├── agents/
│   │   ├── apply_bot.py          # Playwright auto-apply bot; ATS detection and form fill
│   │   ├── tasks.py              # Celery task definitions, Beat schedule, queue routing
│   │   └── scrapers/
│   │       ├── base.py           # Abstract base: rate limiting, dedup, DB persistence
│   │       ├── linkedin.py       # LinkedIn scraper (authenticated)
│   │       ├── indeed.py         # Indeed scraper
│   │       ├── internshala.py    # Internshala scraper
│   │       └── wellfound.py      # Wellfound scraper
│   ├── api/
│   │   └── routes/
│   │       ├── auth.py           # JWT registration, login, /me
│   │       ├── jobs.py           # Job feed: list, filter, get, manual add, trigger analysis
│   │       └── routes.py         # Applications, resumes, cover letters, agent, analytics, chat
│   ├── core/
│   │   ├── config.py             # Pydantic Settings — .env loading, JSON list parsing
│   │   └── database.py           # SQLAlchemy async engine, session factory, get_db dependency
│   ├── models/
│   │   ├── application.py        # Application FSM, ApplicationEvent, ApplicationMethod
│   │   ├── interview.py          # Interview, MockInterviewSession, AgentTask, MarketSnapshot
│   │   ├── job.py                # Job, JobAnalysis, all job enums
│   │   ├── resume.py             # Resume versions, ResumeType
│   │   └── user.py               # User, UserProfile, UserSkill, ExperienceLevel
│   ├── services/
│   │   ├── ai_assistant.py       # Chat service: intent detection, agent action dispatch
│   │   ├── application_service.py# Application CRUD, status transitions
│   │   ├── cover_letter_service.py# GPT-4o cover letter generation
│   │   ├── follow_up_service.py  # Follow-up scheduling and email dispatch
│   │   ├── interview_service.py  # Interview prep, mock sessions, Whisper analysis
│   │   ├── job_analyzer.py       # Two-pass LLM pipeline, match scoring, skill gap extraction
│   │   ├── market_service.py     # Market snapshots, skill demand, salary aggregation
│   │   ├── notification_service.py# Telegram bot + SMTP email dispatch
│   │   └── resume_service.py     # GPT-4o tailoring, ATS keyword injection, PDF rendering
│   ├── schemas/
│   │   └── __init__.py           # All Pydantic v2 request/response schemas
│   ├── utils/
│   │   └── helpers.py            # Shared utility functions
│   └── main.py                   # FastAPI app factory, router registration, lifespan
├── alembic/                       # Database migration scripts
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
├── tests/
├── .env.example                   # Full configuration template
├── alembic.ini
├── requirements.txt               # 40+ pinned dependencies
├── setup.bat                      # Windows first-time setup
└── start_platform.bat             # Windows service launcher
```

---

## Security

### Authentication

All API endpoints require a JWT Bearer token. Register via `POST /api/auth/register`, authenticate via `POST /api/auth/login`. Tokens expire after 24 hours. Passwords are hashed with bcrypt via `passlib`.

### Credential encryption

Platform credentials are encrypted at rest using AES-256-GCM with a key derived via PBKDF2-HMAC-SHA256 (100,000 iterations) and stored as encrypted blobs in the `credential_vaults` table.

### GDPR compliance

A `UserConsent` model tracks granted and revoked consents. Users can request a full data export as JSON (`POST /api/security/data/export`) or submit a deletion request processed within a 30-day window (`POST /api/security/data/delete`).

### Audit logging

All sensitive operations are written to the `AuditLog` table: credential stored/deleted, consent granted/revoked, data export/delete requested, login success/failure, and rate limiting events.

### Secrets management

`SECRET_KEY` and `JWT_SECRET_KEY` must be cryptographically random strings of at least 32 characters. Generate them with:

```bash
openssl rand -hex 32
```

### Auto-apply safeguards

Auto-apply defaults to `disabled`. When enabled, three independent controls limit exposure: the human-in-the-loop Telegram approval gate, a hard daily application limit (`AUTO_APPLY_DAILY_LIMIT`), and a minimum match-score threshold (`AUTO_APPLY_MATCH_THRESHOLD`).

---

## Tech Stack

| Layer | Technology | Version | Role |
|---|---|---|---|
| Web framework | FastAPI | 0.111.0 | REST API, dependency injection, async request handling |
| ASGI server | Uvicorn | 0.30.1 | High-performance async server |
| ORM | SQLAlchemy (async) | 2.0.30 | Async database access, relationship loading |
| Database | PostgreSQL | 15+ | Primary relational datastore |
| Migrations | Alembic | 1.13.1 | Schema version control |
| Task queue | Celery | 5.4.0 | Distributed task execution across four queues |
| Message broker | Redis | 5.0.4 (py) | Celery broker and result backend |
| Scheduler | APScheduler | 3.10.4 | In-process scheduling fallback |
| Queue monitor | Flower | 2.0.1 | Celery task monitoring dashboard |
| LLM (heavy) | OpenAI GPT-4o | openai 1.30.1 | Deep analysis, resume/cover letter generation, interview prep |
| LLM (light) | OpenAI GPT-4o-mini | openai 1.30.1 | Batch scoring, filtering, chat responses |
| Embeddings | text-embedding-3-small | openai 1.30.1 | Semantic profile and job vectorisation |
| Audio transcription | OpenAI Whisper | openai-whisper 20231117 | Interview recording transcription |
| LLM orchestration | LangChain / LangChain-OpenAI | 0.2.1 / 0.1.7 | Prompt chaining, RAG retrieval |
| Vector store | ChromaDB | 0.5.0 | Semantic memory: profile, jobs, resumes |
| Browser automation | Playwright | 1.44.0 | Chromium-based auto-apply bot |
| Async HTTP | httpx | 0.27.0 | Async HTTP requests |
| Concurrent scraping | aiohttp | 3.9.5 | Concurrent async scraping |
| HTML parsing | BeautifulSoup4 + lxml | 4.12.3 / 5.2.2 | Job description extraction |
| Telegram | python-telegram-bot | 21.2 | Bot API, inline keyboards |
| Email | aiosmtplib | 3.0.1 | Async SMTP dispatch |
| Email templates | Jinja2 | 3.1.4 | HTML email rendering |
| PDF generation | WeasyPrint | 62.1 | CSS-to-PDF resume rendering |
| PDF generation | ReportLab | 4.2.0 | Programmatic PDF construction |
| JWT | python-jose | 3.3.0 | JWT signing and verification |
| Password hashing | passlib + bcrypt | 1.7.4 / 4.1.3 | Secure password storage |
| Schema validation | Pydantic v2 | 2.7.1 | Request/response validation |
| Settings | pydantic-settings | 2.3.0 | `.env` loading with type coercion |
| File storage | boto3 (S3) | 1.34.113 | Optional cloud file storage |
| Async file I/O | aiofiles | 23.2.1 | Non-blocking file operations |
| Structured logging | structlog | 24.1.0 | JSON-structured log output |
| Retry logic | tenacity | 8.3.0 | Exponential backoff on transient failures |
| User agent rotation | fake-useragent | 1.5.1 | Scraper bot detection avoidance |
| Frontend | Next.js | 18+ | Dashboard UI |
| Styling | TailwindCSS | 3+ | Utility-first CSS framework |

---

## Roadmap

- [ ] Glassdoor scraper integration
- [ ] S3 storage backend for resume and recording artefacts
- [ ] Frontend Next.js dashboard (job feed, application tracker, interview prep)
- [ ] Salary negotiation assistant powered by market snapshot data
- [ ] Multi-user support with per-user agent isolation
- [ ] Docker Compose deployment configuration
- [ ] Test suite coverage for agent pipeline and service layer
- [ ] Webhook support for ATS status callbacks (Greenhouse, Lever)
- [ ] LangSmith tracing integration for LLM observability

---

## Contributing

Contributions are welcome. Please open an issue to discuss proposed changes before submitting a pull request. Ensure all new code includes type annotations and passes the existing test suite.

---

## License

This project is licensed under the [MIT License](LICENSE).

---

<div align="center">

Built by [Sudharsan Selvaraj](https://github.com/Sudharsanselvaraj)

</div>
