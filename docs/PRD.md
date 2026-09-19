# GroundGuard — Product Requirements Document

## 1. Product Name

**GroundGuard**

### Full description

**GroundGuard — A Low-Latency Sentence-Level Hallucination Detection and Recovery System for LLMs**

---

# 2. Product Vision

GroundGuard is an evidence-grounded AI reliability platform.

Users upload documents, ask questions, and receive AI-generated answers. GroundGuard does not simply trust the generated answer.

It:

1. Retrieves relevant evidence.
2. Generates an answer.
3. Splits the answer into claims.
4. Verifies each claim against evidence.
5. Detects contradiction, entailment, or insufficient support.
6. Optionally attempts recovery for failed claims.
7. Returns the final answer with evidence and grounding information.

The central product idea is:

```text
Retrieve
   ↓
Generate
   ↓
Verify
   ↓
Recover if necessary
   ↓
Return evidence-grounded answer
```

---

# 3. Problem

Retrieval-Augmented Generation improves access to relevant information, but retrieval alone does not guarantee that the generated answer is factually supported by the retrieved evidence.

A model may retrieve:

> Revenue was ₹50 Cr.

but generate:

> Revenue was ₹80 Cr.

GroundGuard addresses this gap by introducing claim-level verification between generation and final response.

---

# 4. Target MVP User Flow

```text
User
 ↓
Create project
 ↓
Upload PDF
 ↓
Wait for ingestion
 ↓
Ask question
 ↓
Retrieve evidence
 ↓
Generate answer
 ↓
Split into claims
 ↓
Verify claims
 ↓
Recover failed claims when enabled
 ↓
Return final answer
 ↓
Display evidence + verification
```

---

# 5. MVP Scope

## Required

### Authentication

* Register
* Login
* Logout
* Current user

### Projects

* Create project
* List projects
* View project
* Update project
* Delete project

### Documents

* Upload PDF
* Track ingestion status
* List project documents
* View document
* Delete document

### RAG

* Parse PDF
* Clean text
* Chunk documents
* Generate embeddings
* Store vectors
* Retrieve relevant chunks
* Construct context

### Generation

* Generate answer using retrieved context
* Split answer into claims
* Attach evidence to claims

### Grounding

* Verify claims against evidence
* Labels:

  * entailment
  * contradiction
  * neutral
* Return grounding score
* Return model version

### UI

* Document upload
* Chat/research interface
* Answer display
* Sources
* Claim-level grounding
* Failed-claim visualization

### Infrastructure

* Docker Compose
* Health checks
* Environment configuration
* Basic logging

---

# 6. MVP Plus

After the basic MVP works:

* Fine-tuned grounding model
* Streaming verification
* Recovery agent
* Reranking
* Hybrid retrieval
* Evaluation dashboard
* Developer API keys
* Improved metrics

---

# 7. Advanced / Research Scope

Not required for MVP:

* KV-cache reuse
* Token-level intervention
* Constrained decoding
* Adaptive retrieval
* Multimodal grounding
* High-performance inference

---

# 8. Core Product Differentiator

GroundGuard is not simply:

```text
RAG chatbot
```

It is:

```text
RAG
+
LLM
+
Claim-level grounding verification
+
Recovery
+
Evidence visualization
```

---

# 9. Example

Source document:

```text
Revenue was ₹50 Cr in 2024.
```

Question:

```text
What was the revenue in 2024?
```

Potential bad generation:

```text
Revenue was ₹80 Cr.
```

Grounding:

```text
Claim:
Revenue was ₹80 Cr.

Evidence:
Revenue was ₹50 Cr.

Result:
Contradiction
```

Recovery:

```text
Retrieve better evidence
 ↓
Regenerate
 ↓
Revenue was ₹50 Cr.
 ↓
Verify
 ↓
Entailment
```

Final result:

```text
Revenue was ₹50 Cr.

✓ Verified
Source: Annual Report, page 12
```

---

# 10. Non-Goals

The MVP is not intended to:

* Build a general-purpose autonomous agent.
* Replace the underlying LLM.
* Guarantee absolute factual correctness.
* Support every document format initially.
* Implement complex multi-agent orchestration.
* Perform frontend-side ML verification.
* Expose internal Python services directly to users.

---

# 11. Success Criteria

The MVP is successful when a user can:

```text
Register
 ↓
Create project
 ↓
Upload PDF
 ↓
Ask question
 ↓
Receive grounded answer
 ↓
Inspect evidence
 ↓
Inspect claim verification
```

and the complete system can demonstrate:

```text
Incorrect claim
 ↓
Detected
 ↓
Recovered
 ↓
Reverified
 ↓
Corrected answer
```
