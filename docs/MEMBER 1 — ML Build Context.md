MEMBER 1 — COMPLETE BUILD CONTEXT
GroundGuard — ML / Grounding Verification Service
1. Your role

You are Member 1 — ML / Grounding Model Engineer.

Your responsibility is to build the machine-learning grounding verification subsystem of GroundGuard.

GroundGuard is an evidence-grounded AI reliability platform. The overall system retrieves evidence, generates an LLM answer, extracts factual claims, and checks whether each claim is supported or contradicted by the retrieved evidence.

Your subsystem is responsible for answering:

“Given this claim and this evidence, does the evidence support the claim, contradict it, or fail to establish it?”

Your work is the ML/scientific core of the project.

You are not responsible for building the RAG pipeline, recovery agent, Node backend, frontend, database application layer, authentication, or deployment.

The project blueprint defines your responsibility as dataset creation, model training, grounding inference, evaluation, and the ML service API.

2. Your ownership boundary

You OWN:

services/ml/
datasets/
ML training
ML preprocessing
ML inference
ML evaluation
ML benchmarks
Model versioning
Grounding score definition
ML service API
ML-specific tests
ML performance measurements

You DO NOT OWN:

PDF parsing
Document ingestion
Chunking pipeline
Embeddings
Vector database
Retrieval
Reranking
LLM generation
Recovery agent
Node.js backend
PostgreSQL application logic
Authentication
Frontend
Docker orchestration
CI/CD

If you need something from another subsystem, consume its contract rather than implementing it yourself.

3. Your exact responsibility

The final ML subsystem must be able to perform:

Claim + Evidence
       ↓
Preprocessing
       ↓
Cross-Encoder
       ↓
Entailment / Contradiction / Neutral
       ↓
Scores
       ↓
Grounding result

Example:

Evidence

Revenue was ₹50 Cr in 2024.

Generated claim

The company generated ₹80 Cr in 2024.

Your service should identify this as:

{
  "label": "contradiction"
}

Conversely:

Evidence

Revenue was ₹50 Cr in 2024.

Claim

The company generated ₹50 Cr in 2024.

Result:

{
  "label": "entailment"
}

And:

Evidence

Revenue was ₹50 Cr in 2024.

Claim

The company opened five new offices in 2024.

Result:

{
  "label": "neutral"
}

The project specification defines these three labels as the initial grounding task.

4. Your service

Your ML service runs independently.

Python
FastAPI
PyTorch
Hugging Face
DeBERTa / RoBERTa
ONNX Runtime later if required

Development URL:

http://localhost:8001

Production/container URL should eventually be:

http://ml:8001

Your service must not depend on the frontend.

Your service must not call the Node backend.

Your service should be independently runnable.

5. Repository ownership

Your primary directory:

services/ml/

Recommended structure:

services/ml/
│
├── src/
│   ├── api/
│   │   ├── routes/
│   │   │   ├── verify.py
│   │   │   ├── batch.py
│   │   │   ├── health.py
│   │   │   └── model.py
│   │   │
│   │   ├── schemas/
│   │   │   ├── verify.py
│   │   │   └── responses.py
│   │   │
│   │   └── app.py
│   │
│   ├── inference/
│   │   ├── model.py
│   │   ├── predictor.py
│   │   └── scoring.py
│   │
│   ├── preprocessing/
│   │   ├── tokenizer.py
│   │   └── normalize.py
│   │
│   ├── evaluation/
│   │   ├── metrics.py
│   │   ├── benchmark.py
│   │   └── calibration.py
│   │
│   └── config.py
│
├── training/
│   ├── train.py
│   ├── dataset.py
│   ├── losses.py
│   └── configs/
│
├── evaluation/
│   ├── datasets/
│   ├── scripts/
│   └── reports/
│
├── models/
│   ├── checkpoints/
│   └── metadata/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
│
├── requirements.txt
├── Dockerfile
└── README.md

You can refine the internal structure, but do not move into another member's directory.

6. First development milestone

Do not immediately start fine-tuning.

Build the ML service in stages.

Stage 1

Get a pretrained NLI baseline running.

POST /verify

works with a pretrained model.

Stage 2

Build a proper dataset.

Stage 3

Benchmark the pretrained model.

Stage 4

Create difficult synthetic contrastive examples.

Stage 5

Fine-tune DeBERTa/RoBERTa.

Stage 6

Compare:

Cosine similarity
Bi-encoder
LLM-as-judge
Pretrained NLI
Fine-tuned cross-encoder
Stage 7

Evaluate latency and calibration.

Stage 8

Expose the final model through the same stable API.

This means the rest of the team can integrate before your research is finished.

7. ML task definition

Your initial classification task is:

INPUT:
Evidence + Claim

OUTPUT:
Entailment
Contradiction
Neutral

Formally:

f(evidence, claim) → {entailment, contradiction, neutral}

The primary model should be a cross-encoder.

Conceptually:

Evidence
   +
Claim
   ↓
Tokenizer
   ↓
DeBERTa / RoBERTa
   ↓
Classification Head
   ↓
3 probabilities

Output:

P(entailment)
P(contradiction)
P(neutral)
8. Important ML principle

Do not assume:

pretrained NLI = factuality detector

It is only your baseline.

The project specifically wants a grounding-oriented model capable of handling RAG-specific factual errors.

Your evaluation must therefore include difficult cases such as:

Number changes
Negation
Entity changes
Dates
Years
Partial support
Unsupported additions
Causal changes

These are specifically identified in the project blueprint.

9. Dataset design

Your fundamental dataset format should be:

{
  "evidence": "...",
  "claim": "...",
  "label": "entailment"
}

or:

{
  "evidence": "...",
  "claim": "...",
  "label": "contradiction"
}

or:

{
  "evidence": "...",
  "claim": "...",
  "label": "neutral"
}

Recommended dataset representation:

id
evidence
claim
label
source
category
difficulty
split

Example:

{
  "id": "sample_001",
  "evidence": "Revenue was ₹50 Cr in 2024.",
  "claim": "Revenue was ₹80 Cr in 2024.",
  "label": "contradiction",
  "source": "synthetic",
  "category": "numerical_swap",
  "difficulty": "hard",
  "split": "train"
}
10. Dataset categories

You should deliberately create examples for:

Normal entailment
Evidence:
The company generated ₹50 Cr.

Claim:
The company generated ₹50 Cr.
Numerical contradiction
Evidence:
The company generated ₹50 Cr.

Claim:
The company generated ₹80 Cr.
Negation
Evidence:
The company did not acquire ABC.

Claim:
The company acquired ABC.
Entity swap
Evidence:
Microsoft acquired Company A.

Claim:
Google acquired Company A.
Date swap
Evidence:
The product launched in 2024.

Claim:
The product launched in 2025.
Unsupported addition
Evidence:
The company generated ₹50 Cr.

Claim:
The company generated ₹50 Cr and employed 2,000 people.
Causal modification
Evidence:
Sales increased after the campaign.

Claim:
The campaign caused the increase in sales.
Partial support
Evidence:
The company operates in India and Singapore.

Claim:
The company operates in India, Singapore, and Japan.
Neutral

Evidence doesn't establish the claim.

These cases should become an important part of your benchmark.

11. Synthetic data

Synthetic data is allowed and useful.

But don't generate only easy examples.

Your synthetic generator should deliberately create minimal factual flips.

For example:

Original:
Revenue was ₹50 Cr.

Generated contradiction:
Revenue was ₹80 Cr.

Only one factual element changes.

This makes the benchmark much more meaningful than random unrelated sentences.

12. Data split rules

Never randomly leak near-duplicates between train and test.

You should maintain:

train
validation
test

And ideally test generalization across:

entities
numbers
documents
domains
templates

For example:

Training:
Company A / Company B

Testing:
Company C / Company D

where appropriate.

The purpose is to determine whether the model learned grounding behavior rather than memorized templates.

13. Baseline experiments

Your research should compare multiple approaches.

At minimum:

Baseline 1 — cosine similarity
Evidence embedding
       +
Claim embedding
       ↓
Cosine similarity
Baseline 2 — bi-encoder

Independent embeddings:

Embedding(evidence)
Embedding(claim)
Baseline 3 — pretrained NLI

Use an existing NLI model without fine-tuning.

Baseline 4 — LLM-as-judge

Use an LLM to determine whether the claim is supported.

Final model

Fine-tuned cross-encoder.

The project blueprint explicitly calls for comparison against cosine similarity, bi-encoder, LLM-as-judge, and your cross-encoder.

14. Metrics

You must report:

Accuracy
Precision
Recall
F1

But especially focus on:

Contradiction recall
False positives
False negatives
Calibration
p50 latency
p95 latency

Why contradiction recall matters:

A hallucinated numerical statement that directly contradicts evidence should not easily pass verification.

15. Grounding score

Your API must expose:

"groundingScore": 0.97

But you must define exactly what it means.

Initial candidate:

groundingScore = P(entailment)

However, treat this as a model score, not automatically as a calibrated probability of truth.

You should evaluate calibration.

Your ML documentation must clearly state:

What groundingScore means
How it is calculated
Whether it is calibrated
How thresholds were selected
16. Decision policy

The model returns probabilities:

{
  "entailment": 0.92,
  "contradiction": 0.03,
  "neutral": 0.05
}

The system needs a label:

entailment

But thresholding must be experimentally defined.

Do not hard-code arbitrary values like:

if score > 0.5:
    PASS

without evaluation.

Instead:

validation dataset
       ↓
threshold experiments
       ↓
precision/recall tradeoff
       ↓
chosen operating threshold
       ↓
documented decision policy
17. API contract

Your service exposes exactly these initial endpoints.

POST /verify
POST /verify/batch
GET  /health
GET  /model/info
POST /evaluate

These endpoints are the ML team's internal service contract.

18. /verify
Request
{
  "requestId": "req_123",
  "claimId": "claim_123",
  "claim": "Revenue was ₹80 Cr.",
  "evidence": [
    {
      "chunkId": "chunk_1",
      "text": "Revenue was ₹50 Cr."
    }
  ]
}
Response
{
  "requestId": "req_123",
  "claimId": "claim_123",
  "label": "contradiction",
  "scores": {
    "entailment": 0.01,
    "contradiction": 0.96,
    "neutral": 0.03
  },
  "groundingScore": 0.01,
  "modelVersion": "grounding-v1"
}

This exact contract is the integration boundary with the rest of the system.

19. Evidence handling

A claim may have multiple evidence chunks.

Example:

{
  "claim": "Revenue was ₹50 Cr in 2024.",
  "evidence": [
    {
      "chunkId": "chunk_1",
      "text": "The company generated ₹30 Cr in Q1."
    },
    {
      "chunkId": "chunk_2",
      "text": "Annual revenue for 2024 was ₹50 Cr."
    }
  ]
}

Your ML service must define how multiple evidence chunks are handled.

Possible strategies include:

Claim vs each evidence
        ↓
Individual scores
        ↓
Aggregation

The exact aggregation strategy is your ML research responsibility.

Document it.

Don't let M2 or M3 independently invent an aggregation strategy.

20. /verify/batch

Purpose:

Efficiently verify multiple claims.

Request:

{
  "requestId": "req_123",
  "claims": [
    {
      "claimId": "claim_1",
      "claim": "Revenue was ₹50 Cr.",
      "evidence": [
        {
          "chunkId": "chunk_1",
          "text": "Revenue was ₹50 Cr."
        }
      ]
    },
    {
      "claimId": "claim_2",
      "claim": "Revenue was ₹80 Cr.",
      "evidence": [
        {
          "chunkId": "chunk_1",
          "text": "Revenue was ₹50 Cr."
        }
      ]
    }
  ]
}

Return one verification result per claim.

This endpoint is useful because M2 may produce multiple claims from one answer.

21. /health

Simple:

GET /health

Response:

{
  "status": "healthy"
}

Later you can include:

model_loaded
model_version
device

but don't make the health endpoint unnecessarily complicated.

22. /model/info

Return:

{
  "modelName": "deberta-grounding",
  "modelVersion": "grounding-v1",
  "labels": [
    "entailment",
    "contradiction",
    "neutral"
  ],
  "device": "cpu"
}

This allows M3 and M4 to know which model produced a result.

23. /evaluate

This is primarily a research endpoint.

It should allow evaluation against a known dataset/configuration.

Conceptually:

{
  "dataset": "grounding-test-v1",
  "modelVersion": "grounding-v1"
}

Response:

{
  "modelVersion": "grounding-v1",
  "metrics": {
    "accuracy": 0.91,
    "precision": 0.90,
    "recall": 0.92,
    "f1": 0.91,
    "contradictionRecall": 0.95
  }
}

Exact fields can evolve, but they must be documented.

24. Model versioning

Every model must have a version.

Examples:

grounding-baseline-v1
grounding-finetuned-v1
grounding-finetuned-v2

Never silently replace:

grounding-v1

with a completely different model.

A model update should create:

new version
new evaluation
new benchmark
25. Model artifact rule

Don't commit massive model checkpoints directly into Git.

Use an agreed model artifact strategy.

Repository should contain:

models/
├── metadata/
│   └── grounding-v1.json
└── checkpoints/

The actual large checkpoint can eventually live in model/object storage.

The metadata should record:

model version
base model
training dataset version
training date
metrics
threshold
tokenizer
configuration
26. Reproducibility requirement

Someone else on the team should be able to understand:

Which dataset?
Which model?
Which hyperparameters?
Which preprocessing?
Which threshold?
Which evaluation?

and reproduce your reported experiment.

Therefore keep:

training/configs/
evaluation/reports/
datasets/version metadata
27. ML logging

Every verification request should be traceable.

At minimum log:

requestId
claimId
modelVersion
latency
label

Do not log sensitive document contents unnecessarily.

28. Performance target

The whole reason this system uses a cross-encoder is that it should be much faster than repeatedly calling an LLM judge.

Therefore measure:

p50 inference latency
p95 inference latency
batch latency
throughput
memory usage

Measure on the actual deployment environment where possible.

Don't claim:

"Real-time"

unless you've actually measured the relevant latency.

29. Unit tests

You should have tests for:

Preprocessing
empty claim
empty evidence
long claim
special characters
numbers
Unicode
Prediction
clear entailment
clear contradiction
clear neutral
API
valid request
missing claim
missing evidence
invalid schema
model unavailable
Batch
one claim
multiple claims
mixed labels
30. Integration tests

Create a test like:

POST /verify
        ↓
model
        ↓
response

and verify that the output exactly matches the agreed schema.

The test should ensure:

requestId preserved
claimId preserved
label valid
scores present
groundingScore present
modelVersion present
31. What M1 receives from M2

M2 will provide evidence in this general structure:

{
  "chunkId": "chunk_123",
  "text": "Revenue was ₹50 Cr.",
  "metadata": {
    "page": 12,
    "source": "annual-report.pdf"
  }
}

M1 does not need to know how M2 generated that evidence.

You consume it.

32. What M1 returns to M2/M3

You return:

label
scores
groundingScore
modelVersion
requestId
claimId

You don't return:

database objects
frontend components
agent decisions
retrieval results

Your service answers one question:

How well is this claim supported by this evidence according to the grounding model?

33. What happens when ML says FAIL?

You don't recover the claim.

That's M2's job.

Your responsibility ends at:

Claim
 ↓
Verify
 ↓
Contradiction / Neutral
 ↓
Return result

Then M2 may:

retrieve more evidence
regenerate
send new claim

back to you.

This separation is critical.

34. Example complete interaction

M2 sends:

{
  "requestId": "req_001",
  "claimId": "claim_001",
  "claim": "The company generated ₹80 Cr in 2024.",
  "evidence": [
    {
      "chunkId": "chunk_001",
      "text": "The company generated ₹50 Cr in 2024."
    }
  ]
}

M1:

DeBERTa
 ↓
Entailment = 0.01
Contradiction = 0.97
Neutral = 0.02

Returns:

{
  "requestId": "req_001",
  "claimId": "claim_001",
  "label": "contradiction",
  "scores": {
    "entailment": 0.01,
    "contradiction": 0.97,
    "neutral": 0.02
  },
  "groundingScore": 0.01,
  "modelVersion": "grounding-v1"
}

M2 then decides:

RECOVER

M2 regenerates:

The company generated ₹50 Cr in 2024.

M2 sends it back to M1.

M1 returns:

{
  "label": "entailment",
  "groundingScore": 0.97
}

Then the claim passes.

35. What you should NOT build

This is extremely important.

Do NOT build:

❌ PDF uploader
❌ PDF parser
❌ Vector database
❌ Embedding pipeline
❌ Retrieval API
❌ Reranker
❌ LLM generation service
❌ Recovery agent
❌ LangGraph workflow
❌ User authentication
❌ Project CRUD
❌ PostgreSQL application schema
❌ Next.js UI
❌ Chat interface
❌ API-key management

If you need any of these, another member owns them.

36. Your communication points with the other members
With M2

You need agreement on:

Evidence schema
Multiple-evidence behavior
Claim schema
Verification timing
Recovery loop
With M3

You need agreement on:

/verify contract
/verify/batch contract
Error schema
requestId
claimId
modelVersion
timeouts
With M4

You need agreement on:

What ML fields should be displayed
Model metrics
Grounding score
Claim status

But M4 should consume the final API rather than directly accessing your Python service.

37. Integration rules specific to M1
Rule 1

Never change /verify response structure silently.

Rule 2

If you need to change it:

Notify M3
Update contract
Update tests
Update documentation
Rule 3

Never expose internal model implementation details through the public API.

Rule 4

Always include:

modelVersion
Rule 5

Always preserve:

requestId
claimId
Rule 6

Never make M2 responsible for interpreting raw model internals.

Return a clean contract.

Rule 7

Don't hard-code thresholds without evaluation.

Rule 8

Every model improvement gets a new evaluation.

38. Your milestone plan
Milestone 1 — Service skeleton

Deliver:

FastAPI
/health
/model/info
/verify

with a pretrained baseline.

Milestone 2 — Dataset

Deliver:

train
validation
test

with:

entailment
contradiction
neutral

and difficult factual perturbations.

Milestone 3 — Baseline evaluation

Produce:

accuracy
precision
recall
F1
contradiction recall
latency
Milestone 4 — Fine-tuning

Train:

DeBERTa/RoBERTa cross-encoder

on the grounding dataset.

Milestone 5 — Threshold/calibration

Determine:

operating thresholds
grounding score behavior
calibration
false positive/negative tradeoffs
Milestone 6 — Optimization

Investigate:

batch inference
ONNX
CPU inference
GPU inference
quantization

only after correctness is established.

Milestone 7 — Integration

Connect:

M2 RAG/Agent
       ↓
M1 ML
       ↓
M3 Backend
       ↓
M4 Frontend

and run the complete E2E flow.

39. Final acceptance criteria for M1

M1 is complete when all of these are true:

[ ] ML service runs independently
[ ] FastAPI service available on :8001
[ ] /health works
[ ] /model/info works
[ ] /verify works
[ ] /verify/batch works
[ ] /evaluate works
[ ] Entailment supported
[ ] Contradiction supported
[ ] Neutral supported
[ ] Difficult factual-flip dataset exists
[ ] Train/validation/test split exists
[ ] Baselines evaluated
[ ] Fine-tuned model evaluated
[ ] Contradiction recall measured
[ ] F1 measured
[ ] Latency measured
[ ] Calibration evaluated
[ ] Grounding score documented
[ ] Model versioning implemented
[ ] Unit tests exist
[ ] API integration tests exist
[ ] Dockerfile exists
[ ] README exists
[ ] API contract matches team contract
[ ] No code from M2/M3/M4 duplicated
[ ] M2 can call /verify successfully
[ ] M3 can integrate the service
40. The mental model M1 should keep throughout development

Your entire responsibility can be reduced to this:

              M2
       Evidence + Claim
               │
               ▼
        ┌───────────────┐
        │   M1 ML       │
        │               │
        │  CrossEncoder │
        │               │
        └───────┬───────┘
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
   ENTAIL   CONTRADICT  NEUTRAL
       │        │        │
       └────────┼────────┘
                ▼
        Grounding Result
                │
                ▼
               M2
          Recovery/Retry
                │
                ▼
               M3
          Persistence/API
                │
                ▼
               M4
               UI

M1's job is not to build GroundGuard. M1's job is to build the reliable grounding-verification brain that the other three members plug into.