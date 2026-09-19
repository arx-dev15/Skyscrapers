# GroundGuard — Development Guide

## 1. Prerequisites

Required:

```text
Git
Node.js
Python
Docker
Docker Compose
PostgreSQL
```

Recommended:

```text
VS Code
```

---

# 2. Repository

```text
grounding-guard/
├── apps/
├── services/
├── packages/
├── infra/
├── datasets/
├── docs/
├── docker-compose.yml
└── README.md
```

---

# 3. Local Services

```text
web       : 3000
api       : 4000
ai        : 8000
ml        : 8001
postgres  : 5432
redis     : 6379
```

---

# 4. Environment

Create:

```text
.env
```

from:

```text
.env.example
```

Never commit `.env`.

---

# 5. Start Infrastructure

```bash
docker compose up --build
```

---

# 6. Development Order

## Phase 1

Foundation:

```text
Repository
Contracts
Database
Docker
Mock AI
Mock ML
```

---

## Phase 2

First vertical slice:

```text
Upload
 ↓
Retrieve
 ↓
Generate
 ↓
Verify
 ↓
Display
```

---

## Phase 3

Real M2:

```text
Parser
Chunking
Embeddings
Vector DB
Retrieval
LLM
```

---

## Phase 4

Real M1:

```text
NLI model
Inference
Thresholds
Evaluation
```

---

## Phase 5

Recovery:

```text
Failed claim
 ↓
Retrieve
 ↓
Regenerate
 ↓
Verify
```

---

## Phase 6

Streaming:

```text
SSE
```

---

## Phase 7

Production:

```text
Docker
CI/CD
Health checks
Monitoring
Deployment
```

This follows the project's recommended development progression.

---

# 7. Branching

Use:

```text
main
```

for stable integration.

Each member uses feature branches.

Example:

```text
m1/grounding-model
m2/rag-pipeline
m3/generation-api
m4/chat-ui
```

---

# 8. Pull Requests

Every PR should contain:

```text
What changed?
Why?
Tests?
Contract changes?
Database changes?
Breaking changes?
```

---

# 9. Testing

Before merging:

```bash
npm test
```

and/or the appropriate Python test command.

Critical integration tests must pass.

---

# 10. E2E Smoke Test

The team must be able to execute:

```text
Register
 ↓
Create project
 ↓
Upload PDF
 ↓
Document READY
 ↓
Ask question
 ↓
Retrieve
 ↓
Generate
 ↓
Verify
 ↓
Persist
 ↓
Display
```

---

# 11. Recovery E2E

Test:

```text
Bad answer
 ↓
Contradiction
 ↓
Recovery
 ↓
New answer
 ↓
Reverification
 ↓
Verified
```

---

# 12. Logging

Every important operation should expose:

```text
requestId
generationId
service
operation
duration
status
```

Example:

```text
[AI] requestId=req_123 operation=retrieve duration=142ms
```

---

# 13. Health Checks

Each service should expose health information.

M1:

```http
GET /health
```

AI:

```http
GET /health
```

API:

```http
GET /health
```

Web should report availability through deployment infrastructure.

---

# 14. Documentation Rule

When implementation changes behavior:

```text
Code
+
Tests
+
Contract
+
Docs
```

must remain synchronized.
