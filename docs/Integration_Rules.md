# GroundGuard — Integration Rules

## Rule 1 — One Project

There is one GroundGuard system.

Do not build four independent applications.

---

# Rule 2 — Ownership

```text
M1 → ML
M2 → AI/RAG/Recovery
M3 → Backend/Database/API
M4 → Frontend/DevOps
```

Do not duplicate functionality.

---

# Rule 3 — Frontend Boundary

Frontend communicates only with:

```text
M3 Node API
```

Never:

```text
M4 → M1
M4 → M2
M4 → PostgreSQL
```

---

# Rule 4 — Backend Boundary

M3 orchestrates:

```text
M3 → M2
M3 → M1
```

M3 owns public API behavior.

---

# Rule 5 — M2/M1 Boundary

M2 asks:

> What evidence is relevant?

M1 asks:

> Does this evidence support this claim?

Therefore:

```text
Retrieval relevance ≠ grounding verification
```

---

# Rule 6 — Every Claim Is Verifiable

Generated claims must have:

```text
claim
evidence
verification
score
modelVersion
```

---

# Rule 7 — Recovery Never Bypasses M1

This is mandatory.

```text
Bad claim
 ↓
Recovery
 ↓
New claim
 ↓
M1
 ↓
PASS
```

Never:

```text
Bad claim
 ↓
Recovery
 ↓
Automatically trust
```

---

# Rule 8 — Recovery Has Limits

Every generation must have:

```text
maxRecoveryAttempts
```

Recovery must terminate on:

```text
PASS
```

or:

```text
MAX_RETRIES
```

or:

```text
TIMEOUT
```

---

# Rule 9 — Project Isolation

Every retrieval operation must be project-scoped.

---

# Rule 10 — Request IDs

Every request:

```text
requestId
```

Every generation:

```text
generationId
```

Every claim:

```text
claimId
```

---

# Rule 11 — Model Versioning

Every ML verification result:

```text
modelVersion
```

---

# Rule 12 — Contract Changes

Before changing an API:

1. Announce.
2. Update contract.
3. Update shared types.
4. Update implementation.
5. Update tests.
6. Update documentation.
7. Run integration tests.

---

# Rule 13 — Mock First

Members must be able to work independently.

Example:

```text
M3
 ↓
Mock M2
 ↓
Mock M1
```

Then:

```text
Real M2
Real M1
```

---

# Rule 14 — Main Branch

`main` must remain stable.

Feature branches:

```text
feature/ml-grounding
feature/rag
feature/backend
feature/frontend
```

---

# Rule 15 — Small PRs

Prefer:

```text
one feature
one focused PR
```

over:

```text
three weeks of changes
one huge PR
```

---

# Rule 16 — No Secrets

Never commit:

```text
.env
API keys
passwords
model credentials
cloud credentials
```

Commit:

```text
.env.example
```

---

# Rule 17 — Testing

Every feature requires tests.

At minimum:

```text
unit
integration
```

Critical flows also require:

```text
E2E
```

---

# Rule 18 — Logs

Logs must contain enough information to trace:

```text
requestId
generationId
service
operation
status
latency
error
```

Do not log secrets.

---

# Rule 19 — Database Ownership

M3 owns application migrations.

M2 owns retrieval implementation.

Shared infrastructure must be coordinated.

---

# Rule 20 — Don't Cross Boundaries

M1 must not suddenly implement retrieval.

M2 must not create public Node endpoints.

M3 must not implement the ML model.

M4 must not implement verification logic.

If a feature crosses boundaries, coordinate before implementing it.

---

# Rule 21 — MVP First

Do not allow advanced research features to block:

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

# Rule 22 — Definition of Integration

A module is not "integrated" because its server starts.

Integration means:

```text
real request
 ↓
real contract
 ↓
real response
 ↓
real persistence
 ↓
real UI behavior
```
