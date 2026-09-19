# GroundGuard — Database Schema

## 1. Database

Primary application database:

```text
PostgreSQL
```

M3 owns schema and migrations.

---

# 2. Entity Relationship

```text
User
 │
 └── Project
       │
       ├── Documents
       │      └── Chunks
       │
       ├── Conversations
       │      └── Messages
       │
       ├── Generations
       │      └── Claims
       │             └── Evidence
       │
       ├── Evaluations
       │
       └── API Keys
```

---

# 3. User

```text
User
```

Fields:

```text
id
email
passwordHash
name
createdAt
updatedAt
```

---

# 4. Project

```text
Project
```

Fields:

```text
id
userId
name
description
createdAt
updatedAt
```

Relationship:

```text
User 1 ─── N Project
```

---

# 5. Document

```text
Document
```

Fields:

```text
id
projectId
name
filePath
mimeType
size
status
createdAt
updatedAt
```

Relationship:

```text
Project 1 ─── N Document
```

---

# 6. Chunk

```text
Chunk
```

Fields:

```text
id
documentId
text
chunkIndex
metadata
createdAt
```

Metadata may contain:

```json
{
  "page": 12,
  "source": "annual-report.pdf"
}
```

Relationship:

```text
Document 1 ─── N Chunk
```

---

# 7. Conversation

```text
Conversation
```

Fields:

```text
id
projectId
title
createdAt
updatedAt
```

---

# 8. Message

```text
Message
```

Fields:

```text
id
conversationId
role
content
createdAt
```

Roles:

```text
user
assistant
system
```

---

# 9. Generation

```text
Generation
```

Fields:

```text
id
requestId
projectId
conversationId
query
answer
status
retrievalLatencyMs
generationLatencyMs
verificationLatencyMs
totalLatencyMs
recoveryAttempts
createdAt
completedAt
```

---

# 10. Claim

```text
Claim
```

Fields:

```text
id
generationId
text
status
label
entailmentScore
contradictionScore
neutralScore
groundingScore
modelVersion
createdAt
updatedAt
```

Relationship:

```text
Generation 1 ─── N Claim
```

---

# 11. Evidence

```text
Evidence
```

Fields:

```text
id
claimId
chunkId
documentId
text
retrievalScore
metadata
createdAt
```

Relationship:

```text
Claim 1 ─── N Evidence
```

---

# 12. Evaluation

```text
Evaluation
```

Fields:

```text
id
projectId
name
status
createdAt
completedAt
```

---

# 13. APIKey

```text
APIKey
```

Fields:

```text
id
projectId
name
keyHash
lastUsedAt
createdAt
revokedAt
```

---

# 14. Important Indexes

At minimum:

```text
Project.userId
Document.projectId
Chunk.documentId
Conversation.projectId
Message.conversationId
Generation.projectId
Generation.requestId
Generation.conversationId
Claim.generationId
Evidence.claimId
Evidence.chunkId
Evaluation.projectId
APIKey.projectId
```

---

# 15. Authorization Rule

Every project-owned entity must ultimately be traceable to:

```text
Project → User
```

M3 must verify ownership before access.

---

# 16. Important Storage Rule

Do not store the entire answer only as one JSON blob.

Store:

```text
Generation
 ↓
Claims
 ↓
Evidence
 ↓
Verification
```

This enables:

* claim-level UI
* evidence inspection
* metrics
* retries
* debugging
* evaluation

This structured claim/evidence approach is part of the original project design.

---

# 17. Vector Storage

Vector storage is M2-owned.

Possible implementations:

```text
pgvector
```

or:

```text
Qdrant
```

If pgvector is used inside the same PostgreSQL deployment, M2 and M3 must coordinate migrations and ownership carefully.
