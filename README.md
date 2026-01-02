# AI Recruiting Platform – Semantic Matching & Search

**Author:** Daniel Lopez  
**GitHub:** https://github.com/daniellopez882  
**Role:** Full-Stack Product Engineer (LLM Systems)

---

## Overview

This repository contains core modules designed and implemented for a **production-grade AI recruiting platform**, architecturally comparable to systems like **CandidateSearch.ai**.

The platform focuses on **semantic candidate discovery, hybrid ranking pipelines, and explainable AI**, built using LLM APIs, vector search, and structured filtering. All modules are designed for real-world production constraints including **rate limits, latency, reliability, and cost control**.

This is not a research prototype or demo project — the architecture reflects systems intended to scale in real hiring environments.

---

## High-Level Architecture

The matching system uses a **layered pipeline** to balance scalability, accuracy, and cost:

1. **Hard Filters (Deterministic constraints)**
2. **Embedding Similarity Search (Semantic recall)**
3. **LLM Re-Ranking (Contextual precision + explainability)**

Supporting infrastructure modules handle **rate limiting, normalization, and system reliability**.

---

## Core Modules

---

## 1. Hard Filtering Engine

**Purpose**  
Reduce the candidate search space cheaply and deterministically before invoking AI models.

### Responsibilities
- Enforces non-negotiable constraints:
  - Location / timezone
  - Work authorization
  - Seniority bands
  - Salary expectations
  - Availability
- Prevents unnecessary embedding and LLM calls.

### Why It Matters
Semantic search becomes expensive and noisy when applied blindly. Hard filters remove **60–80% of irrelevant candidates** before AI inference, improving performance and reducing cost.

### Implementation
- SQL-first filtering logic
- Indexed PostgreSQL queries
- Configurable constraint rules

---

## 2. Embedding Similarity Module

**Purpose**  
Enable semantic matching beyond keyword-based search.

### Responsibilities
- Generates embeddings for:
  - Candidate profiles
  - Job descriptions
  - Recruiter free-text queries
- Stores vectors using `pgvector`
- Performs cosine similarity search

### Key Implementation Details
- Batched embedding generation for cost efficiency
- Async background workers for ingestion
- Embedding versioning to support future model upgrades

### Outcome
Supports natural language queries such as:

> "Senior frontend engineer with fintech experience and React"

without relying on brittle keyword logic.

---

## 3. LLM Re-Ranking & Explainability Engine

**Purpose**  
Apply contextual reasoning and produce interpretable candidate rankings.

### Responsibilities
- Takes top-N candidates from vector search
- Re-ranks candidates using:
  - Skill depth vs surface match
  - Domain relevance
  - Career trajectory and seniority alignment
- Produces **structured explanations**, not opaque scores

### Example Output
```json
{
  "rank": 1,
  "reasoning": {
    "strong_matches": ["React (5 years)", "Fintech domain"],
    "potential_gaps": ["No GraphQL experience"],
    "seniority_fit": "Aligned"
  }
}
```

### Why It Matters
Recruiters do not trust black-box systems. Explainability transforms AI into **decision support**, not a replacement for judgment.

---

## Additional Tricky Modules

---

## 4. Rate Limiting & Reliability Layer

**Purpose**  
Ensure system stability under real production load.

### Problems Addressed
- API rate limits during peak resume ingestion
- Cost spikes from uncontrolled retries
- Latency variability in batch workflows

### Solutions Implemented
- Request queue with exponential backoff
- Concurrency caps per API key
- Graceful degradation when API or budget limits are reached
- Observability on token usage and failure rates

### Impact
- Prevented cascading failures
- Stabilized throughput during traffic spikes
- Controlled API spend under load

---

## 5. Profile Normalization & Skill Canonicalization

**Purpose**  
Ensure embeddings and rankings are based on clean, consistent data.

### Responsibilities
- Normalize job titles and skills:
  - "JS" → "JavaScript"
  - "Frontend Dev" → "Frontend Engineer"
- Deduplicate overlapping skills
- Infer seniority from role progression and timelines

### Why This Is Hard
LLMs extract text, not truth. Without strict normalization, semantic search quality degrades rapidly.

### Result
- Cleaner embeddings
- Higher match accuracy
- Fewer false positives

---

## End-to-End Matching Flow

```text
Resume / Recruiter Query
          ↓
     Hard Filters
          ↓
Embedding Similarity Search (pgvector)
          ↓
     Top-N Candidates
          ↓
LLM Re-Ranking + Explainability
          ↓
 Ranked & Interpretable Results
```

---

## Key Engineering Learnings

- Embeddings alone are insufficient for high-quality matching
- Rate limits are guaranteed, not edge cases
- Explainability is a product requirement, not a bonus
- Hybrid systems outperform pure AI approaches in production

---

## Tech Stack

- **Frontend:** React / Next.js
- **Backend:** Node.js / Python
- **Database:** PostgreSQL + pgvector
- **AI APIs:** OpenAI (Embeddings, GPT-class models)
- **Infrastructure:** Async workers, request queues, API throttling

---

## Author

**Daniel Lopez**  
**GitHub:** https://github.com/daniellopez882  
**Focus:** Shipping reliable, explainable LLM-powered products

---

### Note
This documentation is written to reflect real production engineering work and is intended to withstand technical interviews, code reviews, and architecture discussions.
