# GroundGuard — Architecture Decisions

This document records important project decisions.

Do not use it as a task list.

---

# Decision 001 — Node Is the Public API Gateway

**Status:** Accepted

### Decision

The browser communicates only with the Node backend.

```text
Browser
 ↓
Node
 ↓
Python services
```

### Reason

This centralizes:

* Authentication
* Authorization
* Request validation
* Persistence
* API versioning
* Streaming
* Error handling

---

# Decision 002 — Four Ownership Boundaries

**Status:** Accepted

```text
M1 = Grounding ML

M2 = RAG + LLM + Recovery

M3 = Backend + Platform

M4 = Frontend + DevOps
```

### Reason

Prevents duplicated implementation and unclear responsibility.

---

# Decision 003 — Claims Are First-Class Records

**Status:** Accepted

Generated answers are not stored only as one text blob.

Instead:

```text
Generation
 ↓
Claims
 ↓
Evidence
 ↓
Verification
```

### Reason

Enables claim-level grounding, recovery, metrics and UI.

---

# Decision 004 — M1 Uses NLI-Style Verification

**Status:** Accepted

The grounding model classifies:

```text
entailment
contradiction
neutral
```

### Reason

This provides an explicit claim/evidence relationship rather than relying only on semantic similarity.

---

# Decision 005 — Recovery Must Reverify

**Status:** Accepted

Recovered claims must go through M1 again.

```text
Recovery
 ↓
New claim
 ↓
M1
```

### Reason

Recovery output must not be trusted automatically.

---

# Decision 006 — Recovery Has Maximum Attempts

**Status:** Accepted

Recovery cannot loop indefinitely.

It terminates on:

```text
PASS
MAX_RETRIES
TIMEOUT
```

---

# Decision 007 — Mock Services Are Required

**Status:** Accepted

Members can develop against mock M1/M2 services.

### Reason

Parallel development should not depend on another member finishing first.

---

# Decision 008 — MVP Is Vertical-Slice First

**Status:** Accepted

First prove:

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

before investing heavily in advanced functionality.

---

# Decision 009 — Docker Compose for MVP Integration

**Status:** Accepted

The complete local MVP should run using Docker Compose.

---

# Decision 010 — Shared Contracts Are Frozen

**Status:** Accepted

API changes require coordinated updates.

A member should not silently change:

```text
request body
response body
status values
identifier names
error schema
```

without updating the shared contract.

---

# Decision 011 — Project Isolation Is Mandatory

**Status:** Accepted

Retrieval and all project-owned resources must remain project-scoped.

---

# Decision 012 — Advanced Features Come After MVP

**Status:** Accepted

The following are not allowed to block MVP:

```text
multimodal grounding
token-level intervention
KV-cache optimization
complex multi-agent systems
adaptive retrieval
```
