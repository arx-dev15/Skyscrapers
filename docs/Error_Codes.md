# GroundGuard — Error Codes

## 1. General Format

All API errors:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message"
  },
  "requestId": "req_123"
}
```

---

# 2. Authentication

```text
AUTH_INVALID_CREDENTIALS
AUTH_UNAUTHORIZED
AUTH_SESSION_EXPIRED
AUTH_TOKEN_INVALID
AUTH_USER_NOT_FOUND
```

---

# 3. Authorization

```text
FORBIDDEN
PROJECT_ACCESS_DENIED
DOCUMENT_ACCESS_DENIED
GENERATION_ACCESS_DENIED
```

---

# 4. Validation

```text
VALIDATION_ERROR
INVALID_REQUEST
INVALID_PROJECT_ID
INVALID_DOCUMENT_ID
INVALID_GENERATION_ID
INVALID_CLAIM_ID
```

---

# 5. Projects

```text
PROJECT_NOT_FOUND
PROJECT_CREATE_FAILED
PROJECT_UPDATE_FAILED
PROJECT_DELETE_FAILED
```

---

# 6. Documents

```text
DOCUMENT_NOT_FOUND
DOCUMENT_UPLOAD_FAILED
DOCUMENT_PROCESSING_FAILED
DOCUMENT_UNSUPPORTED_TYPE
DOCUMENT_TOO_LARGE
DOCUMENT_INGESTION_TIMEOUT
```

---

# 7. Retrieval

```text
RETRIEVAL_FAILED
RETRIEVAL_TIMEOUT
VECTOR_STORE_UNAVAILABLE
NO_EVIDENCE_FOUND
```

---

# 8. Generation

```text
GENERATION_FAILED
GENERATION_TIMEOUT
GENERATION_CANCELLED
LLM_UNAVAILABLE
LLM_INVALID_RESPONSE
```

---

# 9. Grounding

```text
VERIFICATION_FAILED
VERIFICATION_TIMEOUT
ML_SERVICE_UNAVAILABLE
MODEL_NOT_LOADED
MODEL_VERSION_NOT_FOUND
```

---

# 10. Recovery

```text
RECOVERY_FAILED
RECOVERY_TIMEOUT
RECOVERY_MAX_ATTEMPTS
RECOVERY_NO_EVIDENCE
```

---

# 11. Database

```text
DATABASE_ERROR
DATABASE_TIMEOUT
DATABASE_CONSTRAINT_ERROR
```

---

# 12. Internal Service Errors

```text
AI_SERVICE_ERROR
ML_SERVICE_ERROR
SERVICE_UNAVAILABLE
SERVICE_TIMEOUT
```

---

# 13. Example

```json
{
  "error": {
    "code": "ML_SERVICE_UNAVAILABLE",
    "message": "Grounding verification service is unavailable"
  },
  "requestId": "req_123"
}
```

---

# 14. Error Handling Rule

Never expose:

```text
stack traces
API keys
database credentials
internal secrets
```

to the frontend.

The backend should translate internal failures into stable public errors.
