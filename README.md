# ExtraContext - Enterprise RAG System

This repository is dedicated to explaining my RAG system.

# PLEASE NOTE THE BACKEND HOSTED ON A LOCAL SERVER, THE FRONTEND IS HOSTED ON VERCEL

## I DID THIS SO NO MATTER WHAT THE LINK WILL NOT BE BROKEN AND U CAN AT LEAST VISIT THE FRONTEND PAGE IF YOU GET AN ERROR LOGGING IN ITS MOST LIKELY BECAUSE THE BACKEND IS DOWN OR IM WORKING ON IT


YOU CAN VISIT THE SITE HERE:  [EXTRACONTEXT](https://www.extracontext.dev)


[![Demo video]([https://youtu.be/e8HGR51ijFc)](https://youtu.be/VIDEO_ID)]


**Note**: This is a private project. This repository contains documentation only to showcase my work for portfolio purposes. The source code remains private.

## Overview

I built a production-ready, multi-tenant Retrieval-Augmented Generation (RAG) system with Google Calendar integration and enterprise-grade authorization. The system allows users to upload PDFs, query their knowledge base semantically, and integrate with Google Calendar for intelligent scheduling queries.

## What I Built

### Core RAG System
- **PDF Processing Pipeline** - Implemented page-aware PDF text extraction and intelligent chunking
- **Vector Search** - Integrated ChromaDB for semantic similarity search with per-tenant data isolation
- **Citation/Provenance** - Built metadata enrichment system to track document sources, page numbers, and content types

- **Smart Query Routing** - Developed keyword-based intent detection to automatically route queries between document search and calendar APIs

### Multi-Tenant Architecture & Security
- **Multi-Tenancy** - Designed complete tenant isolation at both database and vector store levels
- **Two-Layer Authorization**:
  - Scope-based permissions (action-level: can user do X?)
  - ACL-based permissions (resource-level: can user access document Y?)
- **JWT Authentication** - Implemented secure token-based auth with FastAPI OAuth2 scheme
- **Document Sharing** - Built ACL system allowing users to share documents within their tenant

### Google Calendar Integration
- **OAuth 2.0 Flow** - Implemented secure OAuth flow with CSRF protection using HMAC-signed cookies
- **Security-First Design** - Avoided passing JWTs in URLs; used signed HttpOnly cookies for state management
- **Automatic Token Refresh** - Built token manager to automatically refresh expired access tokens using refresh tokens
- **Calendar API Integration** - Integrated Google Calendar API for events and free/busy queries

### Fully Agentic Self-Editing System

The most advanced feature: **the system can autonomously modify its own codebase**. Users submit tasks, and Claude executes them with full read/write access to the entire project.

- **Task Orchestration** - Node.js orchestrator polls for pending tasks and spawns isolated Docker workers
- **Claude CLI Integration** - Workers invoke Claude via a Bridge server, passing the full codebase context
- **Autonomous Execution** - Claude runs with `--dangerously-skip-permissions` for true autonomy
- **Build Verification** - Every change is automatically tested by building a Docker image before merging
- **Git Integration** - Changes are committed to feature branches and merged to main on verification success

**How It Works:**
```
1. User submits task → Backend stores in database
2. Orchestrator polls → Spawns Docker worker container
3. Worker calls Bridge → Bridge proxies to Claude CLI on host
4. Claude modifies code → Commits to worker branch
5. Orchestrator verifies → Builds test Docker image
6. On success → Merges to main branch
```

**The Self-Editing Loop:**
- Claude can modify backend routers, frontend components, the orchestrator itself, worker scripts, and database models
- Each task execution sees the updated codebase from previous tasks
- This creates a recursive improvement loop where Claude improves its own infrastructure

**Architecture Components:**
| Component | Technology | Purpose |
|-----------|------------|---------|
| Orchestrator | Node.js, Dockerode, simple-git | Task dispatch, worker management, git operations |
| Worker | Docker container, Bash | Isolated execution environment |
| Bridge | Node.js Express | Proxies Claude CLI calls (Docker can't run macOS binaries) |
| Task API | FastAPI | CRUD operations, status tracking, execution logs |

### Technical Highlights

**Backend (Python/FastAPI)**
```
- FastAPI for high-performance async API
- SQLAlchemy ORM with SQLite (PostgreSQL-ready)
- ChromaDB for vector embeddings
- Pydantic for type-safe request/response validation
- HMAC-signed cookies for OAuth state management
```

**Frontend (React/Vite)**
```
- React with React Router for SPA
- Custom hooks for API state management
- JWT stored in localStorage with automatic header injection
- Responsive UI with Tailwind CSS
```

**Architecture Decisions**
- Per-tenant ChromaDB collections for data isolation
- Signed cookies (not database) for OAuth state to reduce DB load
- `rsplit` parsing fix for colon-containing values in signed data
- Timezone-aware datetime handling for SQLite compatibility
- Modular router structure (`/integrations/google.py` vs monolithic)

## Key Technical Challenges Solved

### 1. Multi-Tenant Vector Search
**Challenge**: ChromaDB collections were initially per-principal, preventing document sharing.

**Solution**: Refactored to per-tenant collections with ACL-based filtering in the query layer.

### 2. OAuth Cookie State Management
**Challenge**: JWTs in URL query parameters expose tokens in logs and browser history.

**Solution**: Implemented two-step OAuth flow with signed, short-lived HttpOnly cookies containing only principal_id.

### 3. Signed Cookie Parsing Bug
**Challenge**: `rsplit(":", 2)` failed when cookie values contained colons (e.g., `principal_id:state`).

**Solution**: Used `rsplit(":", 1)` to extract signature, then `split(":", 1)` for timestamp/value.

### 4. Timezone-Naive DateTime Comparison
**Challenge**: SQLite returns timezone-naive datetimes, causing comparison errors with `datetime.now(timezone.utc)`.

**Solution**: Added timezone-awareness check and `.replace(tzinfo=timezone.utc)` before comparisons.

### 5. Smart Query Intent Detection
**Challenge**: Users shouldn't need to specify whether they're querying documents or calendar.

**Solution**: Implemented keyword-based intent classifier routing to appropriate backend (RAG vs Calendar API).

### 6. Claude CLI in Docker Containers
**Challenge**: Docker containers on Linux can't execute macOS binaries like the Claude CLI.

**Solution**: Built a Bridge server running on the host that exposes an HTTP endpoint. Workers curl to `host.docker.internal:9999/execute`, and the Bridge invokes Claude CLI locally.

### 7. Safe Autonomous Code Execution
**Challenge**: Allowing AI to modify production code requires safeguards against breaking changes.

**Solution**: Implemented a verification pipeline that builds a test Docker image after every change. Only verified builds get merged to main. Each task runs in an isolated branch (`worker-{taskId}`).

## System Architecture

```
Frontend (React)
    ↓ HTTP + JWT
Backend (FastAPI)
    ├── Auth Layer (JWT, Scopes, ACL)
    ├── RAG Service (ChromaDB, PDF processing)
    ├── Task API (Worker task management)
    └── Integrations (Google OAuth, Calendar API)
    ↓
Database (SQLite)
    ├── Users, Principals, Tenants
    ├── Documents, DocumentACL
    ├── WorkerTasks (execution logs, status, diffs)
    └── ConnectorAccounts (OAuth tokens)
    ↓
Vector Store (ChromaDB)
    └── Per-tenant collections
    ↓
External APIs (Google Calendar)

═══════════════════════════════════════════
         AGENTIC EXECUTION LAYER
═══════════════════════════════════════════

Orchestrator (Node.js)
    ├── Polls Backend for pending tasks
    ├── Spawns Docker workers
    ├── Manages git branches
    └── Verifies builds before merging
    ↓
Worker (Docker Container)
    ├── Isolated execution environment
    ├── Full codebase access (/workspace)
    └── Calls Bridge for Claude CLI
    ↓
Bridge (Node.js on Host)
    ├── HTTP proxy to Claude CLI
    └── Validates workspace paths
    ↓
Claude CLI
    └── Autonomous code modifications
```

## API Highlights

### Authentication
- `POST /api/auth/signup` - Create user account
- `POST /api/auth/login` - Get JWT token

### Documents
- `POST /api/ingest` - Upload PDF (requires `docs:write` scope)
- `GET /api/documents` - List documents (ACL-filtered)
- `DELETE /api/documents/{id}` - Delete document

### Query
- `POST /api/query/smart` - Smart query routing (RAG + Calendar)

### Google Calendar
- `GET /api/integrations/google/prepare` - Start OAuth (sets cookie)
- `GET /api/integrations/google/callback` - OAuth callback
- `GET /api/integrations/google/events` - List calendar events

## Tech Stack

**Backend**
- Python 3.12, FastAPI, SQLAlchemy, Pydantic
- ChromaDB (vector database)
- PyPDF2 (PDF parsing)
- HTTPX (async HTTP client)
- Passlib + bcrypt (password hashing)

**Frontend**
- React 18, Vite, React Router
- Tailwind CSS
- Lucide React (icons)

**Agentic Layer**
- Node.js (Orchestrator & Bridge)
- Dockerode (Docker API for spawning workers)
- simple-git (Git operations)
- Express (Bridge HTTP server)
- Claude CLI (AI execution)

**Infrastructure**
- Docker (isolated worker containers)
- SQLite (development), PostgreSQL-ready
- JWT for authentication
- OAuth 2.0 for third-party integrations
- Git (version control, branch-per-task workflow)

## What I Learned

- Designing multi-tenant systems with proper data isolation
- Implementing two-layer authorization (scopes + ACLs)
- Securing OAuth flows without exposing tokens
- Building production-ready APIs with FastAPI
- Vector search optimization and metadata filtering
- Handling timezone-aware datetime operations
- Debugging cookie-based state management
- Building autonomous AI agent systems with proper isolation
- Docker container orchestration and the Dockerode API
- Bridging macOS host services into Linux containers
- Designing verification pipelines for AI-generated code
- Git workflow automation (branch-per-task, auto-merge)

## Future Enhancements

If I were to continue this project, I would:
- Implement Notion and Slack integrations following the Google Calendar pattern
- Add database migrations with Alembic
- Implement token encryption at rest
- Add comprehensive test suite (pytest)
- Deploy to production with PostgreSQL + Redis
- Add rollback capability for failed agentic changes
- Implement approval workflows for high-risk modifications

---

**Status**: Active private project demonstrating full-stack development, system design, and AI agent architecture.

**Timeline**: Built over multiple sessions with iterative improvements.

**Purpose**: Personal knowledge management system with enterprise-grade security, multi-tenant architecture, and fully autonomous self-editing capabilities.
