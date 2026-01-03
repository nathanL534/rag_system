

This repository is dedicated to explaining my RAG system. 

***PLEASE NOTE THE BACKEND IS THE ONLY THING HOSTED LOCALLY, THE FRONTEND IS HOSTED ON VERCEL, I DID THIS SO NO MATTER WHAT THE LINK WILL NOT BE BROKEN AND U CAN AT LEAST VISIT THE FRONTEND PAGE IF YOU GET AN ERROR LOGGING IN ITS MOST LIKELY BECAUSE THE BACKEND IS DOWN OR IM WORKING ON IT***


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

## System Architecture

```
Frontend (React)
    ↓ HTTP + JWT
Backend (FastAPI)
    ├── Auth Layer (JWT, Scopes, ACL)
    ├── RAG Service (ChromaDB, PDF processing)
    └── Integrations (Google OAuth, Calendar API)
    ↓
Database (SQLite)
    ├── Users, Principals, Tenants
    ├── Documents, DocumentACL
    └── ConnectorAccounts (OAuth tokens)
    ↓
Vector Store (ChromaDB)
    └── Per-tenant collections
    ↓
External APIs (Google Calendar)
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

**Infrastructure**
- SQLite (development), PostgreSQL-ready
- JWT for authentication
- OAuth 2.0 for third-party integrations

## What I Learned

- Designing multi-tenant systems with proper data isolation
- Implementing two-layer authorization (scopes + ACLs)
- Securing OAuth flows without exposing tokens
- Building production-ready APIs with FastAPI
- Vector search optimization and metadata filtering
- Handling timezone-aware datetime operations
- Debugging cookie-based state management

## Future Enhancements

If I were to continue this project, I would:
- Add LLM integration for answer generation (currently retrieval-only)
- Implement Notion and Slack integrations following the Google Calendar pattern
- Add database migrations with Alembic
- Implement token encryption at rest
- Add comprehensive test suite (pytest)
- Deploy to production with PostgreSQL + Redis

---

**Status**: Private project built to demonstrate full-stack development and system design skills.

**Timeline**: Built over multiple sessions with iterative improvements.

**Purpose**: Personal knowledge management system with enterprise-grade security and multi-tenant architecture.
