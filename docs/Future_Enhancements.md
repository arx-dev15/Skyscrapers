# GroundGuard — Future Enhancements & Post-MVP Roadmap

## 1. Purpose

This document defines capabilities that may be added **after the core GroundGuard MVP is stable**.

These features are intentionally separated from the MVP so that the team does not over-engineer the first release.

The principle is:

```text
MVP
 ↓
Stable architecture
 ↓
Measure
 ↓
Identify bottlenecks
 ↓
Add improvements
 ↓
Research / advanced capabilities
```

---

# 2. Current MVP Boundary

The MVP should first provide:

```text
PDF ingestion
      ↓
Chunking
      ↓
Embeddings
      ↓
Retrieval
      ↓
LLM generation
      ↓
Claim segmentation
      ↓
ML grounding
      ↓
Evidence display
      ↓
Node API
      ↓
Next.js UI
      ↓
Dockerized deployment
```

Only after this flow is reliable should the team expand the system.

---

# 3. Post-MVP Feature Groups

Future work is divided into:

```text
Phase A — MVP+
Phase B — Production Intelligence
Phase C — Advanced AI Reliability
Phase D — Research
Phase E — Platform Expansion
```

---

# PHASE A — MVP+

These are the first enhancements after the core MVP.

---

# A1. Fine-Tuned Grounding Model

### Current

Start with:

```text
Pretrained NLI model
```

### Enhancement

Fine-tune a cross-encoder specifically for GroundGuard's grounding task.

Architecture:

```text
Claim
 +
Evidence
 ↓
DeBERTa / RoBERTa
 ↓
Entailment
Contradiction
Neutral
```

Training examples should emphasize difficult cases:

```text
Numerical swaps
Negation
Entity swaps
Dates
Unsupported additions
Causal changes
Partial support
```

The blueprint specifically identifies these difficult examples as valuable for the grounding dataset.

---

# A2. Grounding Calibration

A raw entailment probability should not automatically be treated as factual confidence.

Add:

```text
Calibration
 ↓
Validation analysis
 ↓
Threshold selection
 ↓
Better PASS / FAIL decisions
```

Track:

```text
Accuracy
Precision
Recall
F1
Contradiction recall
False positives
Calibration
```

---

# A3. Streaming Verification

Instead of waiting for the entire answer:

```text
LLM
 ↓
Sentence
 ↓
Verify
```

The system can progressively verify generated sentences.

Example:

```text
Sentence 1 → verified
Sentence 2 → verified
Sentence 3 → flagged
Sentence 4 → verifying
```

Frontend receives events through SSE.

---

# A4. Recovery Agent

Add the full recovery loop:

```text
Generated Claim
       ↓
Verification
       ↓
FAIL
       ↓
Diagnose
       ↓
Retrieve more evidence
       ↓
Regenerate
       ↓
Verify again
```

Recovery must retain:

```text
maxRecoveryAttempts
timeout
termination condition
```

The recovery design should remain deterministic initially rather than becoming a large multi-agent system.

---

# A5. Hybrid Retrieval

Current:

```text
Vector search
```

Enhancement:

```text
Vector search
      +
Keyword search
      ↓
Hybrid ranking
```

Potential pipeline:

```text
Query
 ↓
Dense retrieval
 +
Sparse retrieval
 ↓
Merge
 ↓
Rerank
 ↓
Context
```

---

# A6. Better Reranking

Current retrieval can be improved using a dedicated reranker.

Important distinction:

```text
Reranker
=
Question ↔ Evidence relevance
```

while:

```text
Grounding model
=
Claim ↔ Evidence support
```

Do not merge these responsibilities.

---

# A7. Retrieval Evaluation

Introduce measurable retrieval quality.

Track:

```text
Recall@K
Precision@K
MRR
NDCG
Evidence coverage
```

This helps determine whether a grounding failure is caused by:

```text
Bad retrieval
```

or:

```text
Bad generation
```

or:

```text
Bad verification
```

---

# PHASE B — PRODUCTION INTELLIGENCE

After the MVP+ functionality works.

---

# B1. Query Rewriting

If the user's question is ambiguous:

```text
User Query
 ↓
Query Rewriter
 ↓
Improved Retrieval Query
 ↓
RAG
```

Potential strategies:

```text
Query expansion
Query decomposition
Keyword extraction
Entity extraction
Question reformulation
```

---

# B2. Adaptive Retrieval

Instead of always retrieving the same number of chunks:

```text
Simple query
 → retrieve fewer chunks

Complex query
 → retrieve more chunks
```

Possible loop:

```text
Query
 ↓
Retrieve
 ↓
Evaluate evidence coverage
 ↓
Enough?
 ├── YES → Generate
 └── NO  → Retrieve again
```

---

# B3. Multi-Hop Retrieval

For questions requiring multiple pieces of evidence:

```text
Question
 ↓
Evidence A
 ↓
New sub-question
 ↓
Evidence B
 ↓
Combine
 ↓
Generate
```

Example:

```text
Who founded Company X?

Evidence A:
Company X was founded by Person A.

Evidence B:
Person A previously founded Company Y.
```

---

# B4. Evidence Conflict Resolution

If documents disagree:

```text
Document A
"Revenue = ₹50 Cr"

Document B
"Revenue = ₹55 Cr"
```

GroundGuard should detect the conflict rather than silently selecting one.

Potential result:

```text
⚠ Conflicting evidence

Source A → ₹50 Cr
Source B → ₹55 Cr

Reason:
Sources disagree.
```

The recovery architecture already identifies comparing conflicting evidence as a potential recovery tool.

---

# B5. Source Reliability

Introduce optional source metadata:

```text
sourceType
publicationDate
authority
documentVersion
```

Then evidence can be displayed with richer context.

Important:

```text
Source reliability
≠
Grounding verification
```

The two should remain separate signals.

---

# B6. Document Versioning

Support:

```text
Annual Report 2024
Annual Report 2025
Annual Report 2026
```

and allow the system to distinguish:

```text
latest
historical
superseded
```

This becomes important for time-sensitive questions.

---

# B7. Better Observability

Track every generation:

```text
requestId
generationId

retrieval latency
reranking latency
LLM latency
verification latency
recovery latency
total latency

number of claims
number of failed claims
number of recovery attempts
```

Then create a full trace:

```text
Request
 ↓
Retrieval
 ↓
Generation
 ↓
Verification
 ↓
Recovery
 ↓
Final response
```

---

# B8. Evaluation Dataset

Build a permanent GroundGuard benchmark.

Dataset structure:

```text
Question
Evidence
Expected claim
Generated claim
Grounding label
Recovery result
```

Include difficult cases:

```text
Numbers
Dates
Negation
Entities
Causal statements
Partial evidence
Unsupported claims
Conflicting evidence
```

---

# B9. Regression Testing

Every new model or retrieval change should run against the benchmark.

Example:

```text
grounding-v1
      ↓
Benchmark
      ↓
Metrics

grounding-v2
      ↓
Benchmark
      ↓
Metrics
```

Do not deploy a new model simply because it performs well on one example.

---

# PHASE C — ADVANCED AI RELIABILITY

These capabilities move GroundGuard beyond a basic RAG + verification system.

---

# C1. Claim Decomposition

Complex statements can be broken into atomic claims.

Example:

```text
"The company earned ₹50 Cr in 2024,
grew 20%, and opened 5 offices."
```

becomes:

```text
Claim 1:
Revenue = ₹50 Cr

Claim 2:
Growth = 20%

Claim 3:
Opened 5 offices
```

Each claim receives independent evidence and verification.

---

# C2. Evidence Coverage Analysis

Instead of simply checking whether a claim is supported:

```text
Claim
 ↓
Which parts are supported?
```

Example:

```text
"The company generated ₹50 Cr
and grew 20%."

Evidence:
Revenue → supported
Growth → unsupported
```

Result:

```text
Partial support
```

---

# C3. Fine-Grained Grounding

Move beyond sentence-level grounding toward:

```text
Sentence
 ↓
Phrase
 ↓
Token/span
 ↓
Evidence
```

Potential UI:

```text
Revenue was ₹50 Cr
^^^^^^^^^^^^
Supported by page 12
```

---

# C4. Token-Level Intervention

Instead of waiting until a complete sentence is generated:

```text
Token generation
 ↓
Risk detection
 ↓
Intervention
```

This is an advanced research capability.

The original blueprint identifies token-level intervention as a research/advanced direction.

---

# C5. Constrained Decoding

Use retrieved evidence to constrain generation.

Conceptually:

```text
Evidence
 ↓
Generation constraints
 ↓
LLM
 ↓
More grounded answer
```

This would complement post-generation verification rather than necessarily replacing it.

---

# C6. Adaptive Verification

Not every claim needs identical verification cost.

Example:

```text
Low-risk claim
 → fast verification

High-risk / ambiguous claim
 → deeper verification
```

Potential factors:

```text
Confidence
Evidence count
Claim complexity
Numerical content
Conflicting sources
Historical sensitivity
```

---

# C7. Verification Cascades

Use multiple verification layers:

```text
Fast similarity check
        ↓
Cross-encoder
        ↓
Deep verifier
```

Only expensive verification is triggered when necessary.

Potential architecture:

```text
Claim
 ↓
Cheap verifier
 ↓
Confident?
 ├── YES → result
 └── NO
      ↓
Cross-encoder
      ↓
Confident?
 ├── YES → result
 └── NO
      ↓
Deep verification
```

---

# C8. Specialized Verifiers

Different claim types could use different verification strategies.

Examples:

```text
Numerical claim
 → numerical consistency verifier

Date claim
 → temporal verifier

Citation claim
 → source verification

Entity claim
 → entity consistency verifier
```

---

# C9. Numerical Reasoning Verification

For claims such as:

```text
Revenue increased by 20%.
```

verify:

```text
Previous value
Current value
Calculated percentage
Generated statement
```

This could detect:

```text
Arithmetic errors
Percentage errors
Unit mismatches
Currency mismatches
```

---

# C10. Temporal Grounding

Detect:

```text
2024 revenue
```

being incorrectly used to answer:

```text
2026 revenue
```

Potential verification dimensions:

```text
Entity
Date
Value
Source
Claim
```

---

# PHASE D — RESEARCH / HIGH-PERFORMANCE

These are longer-term research directions.

---

# D1. KV-Cache Reuse

Optimize repeated model computation during:

```text
Generation
Verification
Recovery
```

Goal:

```text
Lower latency
Lower compute cost
```

---

# D2. Advanced Inference Optimization

Potential techniques:

```text
Model quantization
ONNX optimization
Batch inference
Dynamic batching
GPU optimization
Model caching
```

---

# D3. High-Performance Grounding Service

Scale M1 independently:

```text
             ┌── ML Worker
API → Queue ─┼── ML Worker
             ├── ML Worker
             └── ML Worker
```

Potentially support:

```text
batch verification
parallel claim verification
GPU inference
```

---

# D4. Distributed Recovery

Instead of sequential recovery:

```text
Claim
 ↓
Retrieve
 ↓
Regenerate
```

potentially explore:

```text
                 Claim
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Search A   Search B   Search C
        │          │          │
        └──────────┼──────────┘
                   ▼
                Combine
                   ↓
                Verify
```

This should only be considered after the deterministic recovery architecture is stable.

---

# D5. Multimodal Grounding

Support:

```text
PDF text
Images
Tables
Charts
Scanned documents
```

Potential pipeline:

```text
Document
 ↓
Text extraction
+
Image extraction
+
Table extraction
 ↓
Multimodal evidence
 ↓
Generation
 ↓
Grounding
```

Multimodal grounding is explicitly identified as an advanced direction in the original blueprint.

---

# D6. Table Grounding

For financial and analytical documents:

```text
Question
 ↓
Find table
 ↓
Identify row/column
 ↓
Extract value
 ↓
Verify claim
```

Example:

```text
Revenue | 2024 | ₹50 Cr
```

---

# D7. Chart Grounding

Support questions such as:

> What was the highest revenue year in this chart?

The system should identify:

```text
Chart
 ↓
Data points
 ↓
Relevant evidence
 ↓
Claim
 ↓
Verification
```

---

# D8. Agentic Research Mode

Move from:

```text
Question
 ↓
One retrieval
 ↓
Answer
```

to:

```text
Question
 ↓
Plan
 ↓
Search
 ↓
Analyze
 ↓
Search again
 ↓
Compare
 ↓
Verify
 ↓
Answer
```

This is substantially more complex and should come after the basic recovery agent.

---

# PHASE E — PLATFORM EXPANSION

---

# E1. Developer API

Expose GroundGuard as a developer service.

Example:

```text
POST /v1/projects/:projectId/generations
```

Developers can integrate:

```text
GroundGuard verification
```

into their own applications.

The original project already identifies a developer platform/API-key layer as part of the broader product.

---

# E2. SDKs

Potential SDKs:

```text
Python
TypeScript / JavaScript
```

Example conceptual usage:

```text
groundguard.generate(...)
```

---

# E3. Webhooks

Allow external applications to receive events:

```text
generation.completed
generation.failed
claim.flagged
recovery.completed
```

---

# E4. Batch Evaluation API

Allow users to submit:

```text
100 questions
```

and receive:

```text
grounding rate
contradiction rate
retrieval metrics
latency
recovery statistics
```

---

# E5. Model Comparison

Evaluate multiple grounding models:

```text
grounding-v1
grounding-v2
grounding-v3
```

Compare:

```text
Accuracy
F1
Contradiction recall
Latency
Calibration
```

---

# E6. Retrieval Comparison

Compare:

```text
Dense
Sparse
Hybrid
Reranked
```

against the same benchmark.

---

# E7. Evaluation Dashboard

Eventually provide:

```text
┌─────────────────────────────────────┐
│ Grounding Rate          94.2%       │
│ Contradiction Rate       3.1%       │
│ Recovery Success        81.5%       │
│ Avg Retrieval          120ms        │
│ Avg Verification        42ms        │
│ Avg Generation         1.8s         │
└─────────────────────────────────────┘
```

---

# E8. Audit / Trace View

Allow users to inspect:

```text
Question
 ↓
Retrieved evidence
 ↓
Generated claim
 ↓
Verification
 ↓
Recovery
 ↓
Final claim
```

This becomes especially useful for enterprise applications.

---

# E9. Enterprise Access Controls

Potential additions:

```text
Organizations
Teams
Roles
Permissions
Shared projects
Private projects
Audit logs
```

---

# E10. External Knowledge Sources

Future integrations may include:

```text
URLs
Web pages
Knowledge bases
Cloud storage
Enterprise document systems
Databases
```

Each source should still eventually enter the same evidence-grounding architecture.

---

# 4. Future Architecture Evolution

The MVP architecture:

```text
M4
 ↓
M3
 ↓
M2
 ↓
M1
```

can evolve into:

```text
                         USER
                           │
                           ▼
                         M4
                           │
                           ▼
                         M3
                           │
              ┌────────────┼─────────────┐
              │            │             │
              ▼            ▼             ▼
          Retrieval     Generation    Verification
              │            │             │
              └────────────┼─────────────┘
                           │
                           ▼
                    Recovery Engine
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        Fast Verify    Deep Verify   Specialized
                                      Verifiers
                           │
                           ▼
                     Evidence Graph
```

The key principle is that future capabilities should **extend existing boundaries**, not destroy them.

---

# 5. Future Model Architecture

Current:

```text
Claim + Evidence
       ↓
NLI Cross Encoder
       ↓
Label
```

Future:

```text
Claim
 │
 ├── Semantic verification
 ├── Numerical verification
 ├── Temporal verification
 ├── Entity verification
 ├── Source verification
 └── Evidence coverage
       │
       ▼
   Grounding Engine
       │
       ▼
Unified Grounding Result
```

---

# 6. Future Grounding Result

Potential future schema:

```json
{
  "claimId": "claim_1",
  "overallStatus": "verified",
  "groundingScore": 0.96,
  "checks": {
    "semantic": {
      "status": "pass",
      "score": 0.97
    },
    "numerical": {
      "status": "pass"
    },
    "temporal": {
      "status": "pass"
    },
    "entity": {
      "status": "pass"
    }
  },
  "evidence": []
}
```

This should be treated as a future evolution, not an MVP requirement.

---

# 7. Future Performance Goals

As the system matures, measure:

```text
Retrieval latency
Generation latency
Verification latency
Recovery latency
End-to-end latency
Throughput
GPU utilization
CPU utilization
Memory
Cost per generation
```

Optimization should be driven by actual measurements rather than premature optimization.

---

# 8. Future Reliability Metrics

Eventually track:

```text
Grounding accuracy
Contradiction recall
Unsupported-claim rate
False-positive rate
Recovery success rate
Evidence coverage
Retrieval recall
Calibration error
Average recovery attempts
```

---

# 9. Future Product Modes

Potential future modes:

### Chat Mode

```text
Ask → Answer → Evidence
```

### Research Mode

```text
Ask
 ↓
Multi-step retrieval
 ↓
Evidence synthesis
 ↓
Verification
 ↓
Detailed answer
```

### Verification Mode

```text
User provides answer
 ↓
GroundGuard checks claims
 ↓
Evidence
 ↓
Verification report
```

### Developer Mode

```text
Application
 ↓
GroundGuard API
 ↓
Verification result
```

### Evaluation Mode

```text
Dataset
 ↓
Generate
 ↓
Verify
 ↓
Metrics
 ↓
Model comparison
```

---

# 10. Future Research Questions

The team can eventually investigate:

```text
Can verification happen during generation?

Can retrieval adapt based on verification failures?

Can grounding be performed at token/span level?

Can recovery choose evidence automatically?

Can different claim types use specialized verifiers?

Can verification latency be reduced without sacrificing recall?

Can grounding confidence be reliably calibrated?

Can the system identify why a claim failed?

Can the system distinguish retrieval failure from generation failure?
```

These become research directions rather than MVP requirements.

---

# 11. Future Development Principle

Every future feature should answer at least one of these:

```text
Does it improve grounding accuracy?

Does it reduce hallucination?

Does it improve evidence retrieval?

Does it reduce verification latency?

Does it improve recovery?

Does it improve observability?

Does it improve developer usability?

Does it improve scalability?
```

If not, it should not automatically become a priority.

---

# 12. Future Feature Priority Framework

Use:

```text
P0 = MVP blocker
P1 = immediate post-MVP
P2 = production enhancement
P3 = research
```

Current examples:

```text
P0
PDF
RAG
LLM
Grounding
UI
Docker

P1
Fine-tuned model
Streaming verification
Recovery
Reranking
Evaluation

P2
Hybrid retrieval
Query rewriting
Adaptive retrieval
Conflict resolution
Document versioning
Developer SDK
Observability

P3
Token intervention
Constrained decoding
KV-cache optimization
Multimodal grounding
Specialized verifiers
Advanced agentic research
```

---

# 13. Important Rule

Future features must not break the core GroundGuard principle:

```text
Evidence
   ↓
Generation
   ↓
Claim
   ↓
Verification
   ↓
Recovery if required
   ↓
Verified result
```

Even as the system becomes more sophisticated, this remains the central architecture.

---

# 14. Final Roadmap

```text
                    GROUNDGUARD
                         │
                         ▼
                       MVP
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
           MVP+                  Production
              │                     │
       ┌──────┼──────┐       ┌──────┼──────┐
       ▼      ▼      ▼       ▼      ▼      ▼
    Fine ML Recovery SSE   Hybrid  Adaptive Conflict
       │      │      │       │      │      │
       └──────┴──────┘       └──────┴──────┘
              │                     │
              └──────────┬──────────┘
                         ▼
                  Advanced Reliability
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          Token       Multimodal   Specialized
       Intervention   Grounding    Verifiers
             │           │           │
             └───────────┼───────────┘
                         ▼
                   Research Platform
                         │
                         ▼
                 Developer Platform
```

---

# 15. Guiding Principle

**Do not build the future before proving the MVP.**

The MVP establishes the foundation:

```text
RAG
+
LLM
+
Claim-level ML verification
+
Evidence
+
Node orchestration
+
Next.js UI
```

Future work should make that foundation:

```text
more accurate
more reliable
faster
more explainable
more scalable
more programmable
```

without destroying the ownership and service boundaries established for the MVP.
