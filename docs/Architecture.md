# GroundGuard — System Architecture

## 1. Architecture Overview

GroundGuard is a four-module system.

```text
                    USER
                      │
                      ▼
              ┌───────────────┐
              │   M4 WEB      │
              │   Next.js     │
              └───────┬───────┘
                      │
                  REST + SSE
                      │
                      ▼
              ┌───────────────┐
              │   M3 API      │
              │ Node + TS     │
              └───────┬───────┘
                      │
             ┌────────┼─────────┐
             │        │         │
             ▼        ▼         ▼
           M2 AI    M1 ML    PostgreSQL
             │        │
             │        │
             ▼        ▼
         Vector DB   Model
```

---

# 2. Services

| Service    | Owner | Technology           |     Port |
| ---------- | ----- | -------------------- | -------: |
| Web        | M4    | Next.js + TypeScript |     3000 |
| API        | M3    | Node.js + TypeScript |     4000 |
| AI         | M2    | Python + FastAPI     |     8000 |
| ML         | M1    | Python + FastAPI     |     8001 |
| PostgreSQL | M3    | PostgreSQL           |     5432 |
| Redis      | M3    | Redis                |     6379 |
| Vector DB  | M2    | pgvector/Qdrant      | internal |

The overall source architecture uses Next.js → Node → Python services, with PostgreSQL/Redis supporting the backend.

---

# 3. Ownership

## M1 — Grounding ML

Owns:

* NLI dataset
* Grounding model
* Model training
* Model evaluation
* Inference API
* Model versioning
* Grounding thresholds

Does not own:

* Retrieval
* LLM
* Node API
* Database
* Frontend

---

## M2 — AI/RAG

Owns:

* Document parsing
* Chunking
* Embeddings
* Vector search
* Retrieval
* Reranking
* Context construction
* LLM generation
* Claim processing
* Recovery agent

Does not own:

* Public Node API
* Application database
* Frontend
* Grounding model

---

## M3 — Backend/Platform

Owns:

* Public API
* Authentication
* Authorization
* Database
* Request orchestration
* Persistence
* SSE
* Cancellation
* API keys
* Metrics
* Evaluation endpoints

---

## M4 — Frontend/DevOps

Owns:

* Next.js UI
* Chat UI
* Evidence UI
* Claim UI
* Evaluation UI
* Docker integration
* CI/CD
* Monitoring
* Deployment
* E2E frontend experience

---

# 4. Communication Rules

Frontend:

```text
M4 → M3 ONLY
```

Backend:

```text
M3 → M2
M3 → M1
M3 → PostgreSQL
M3 → Redis
```

AI:

```text
M2 → Vector DB
M2 → LLM
M2 → M1
```

The browser must never directly access:

```text
M1
M2
PostgreSQL
Vector DB
Redis
```

---

# 5. Main Generation Architecture

```text
M4
 ↓
M3
 ↓
M2 Retrieval
 ↓
M2 LLM
 ↓
Claim Processor
 ↓
M1 Verification
 ↓
 ┌──────────────┐
 │              │
PASS           FAIL
 │              │
 │              ▼
 │          M2 Recovery
 │              │
 │              ▼
 │          M1 Verify
 │              │
 └───────┬──────┘
         ▼
       M3
         ↓
       M4
```

---

# 6. Document Architecture

```text
Upload
 ↓
M3 Document
 ↓
M2 Ingestion
 ↓
Parser
 ↓
Cleaner
 ↓
Chunker
 ↓
Embedding
 ↓
Vector Store
```

---

# 7. Project Isolation

Every retrieval request must include:

```text
projectId
```

Retrieval must never return chunks belonging to another project.

---

# 8. Request Tracing

Every request gets:

```text
requestId
```

Every generation gets:

```text
generationId
```

Every claim gets:

```text
claimId
```

Every ML result gets:

```text
modelVersion
```

This allows:

```text
Frontend
 ↓
requestId
 ↓
Node logs
 ↓
AI logs
 ↓
ML logs
```

---

# 9. Data Ownership

M3 owns the application database.

M2 owns the retrieval/vector layer.

M1 owns model artifacts and ML evaluation artifacts.

M4 owns deployment configuration and UI.

If pgvector is used inside PostgreSQL, M2 and M3 must coordinate because the storage infrastructure is shared.

---

# 10. Failure Isolation

If M1 fails:

```text
generation.status = verification_failed
```

If M2 fails:

```text
generation.status = generation_failed
```

If document ingestion fails:

```text
document.status = failed
```

No service should silently pretend another service succeeded.
