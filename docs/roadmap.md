# Roadmap

Repository initialization precedes Phase 1. Each phase adds only the dependencies it needs. Production code, notebook experiments, fixtures, and automated tests live in their designated directories.

## Approval and sequencing

Only repository initialization is complete. [P1-001](tasks/phase-01-document-extraction.md) is proposed and awaits explicit approval; it is not ready for development. Later phases are planning targets, not authorization to install dependencies or provision resources.

The core dependency chain is Phase 1 → 2 → 3 → 4. Phase 5 consumes the tested Phase 4 workflow as a tool; Phase 6 adds state to that agent. Phase 7 requires the core RAG pipeline; it does not inherently require agents or long-term memory. The current educational order keeps Phases 5–6 before the UI, without making them application prerequisites. Phase 8 follows the local application and ingestion contracts. Phase 10 requires an evaluated single-agent baseline and a demonstrated task that benefits from coordination.

Testing, attribution, diagnostics, and small repeatable evaluation sets start in Phase 1. Add labeled retrieval queries in Phase 3, grounded-answer checks in Phase 4, and tool-use checks in Phase 5. Phase 9 consolidates operational hardening; it does not postpone correctness or security until then. Document-level authorization must be defined and enforced at upload, retrieval, and source access before Phase 7's multiuser acceptance, independently of authentication. Treat retrieved text as untrusted input when generation/tools are introduced.

## Phase targets

| Phase | Scope | Acceptance criteria |
| --- | --- | --- |
| 1. Document processing | Local PDF/TXT extraction and metadata-preserving character chunking; see proposed P1-001. | Pass source/offset/determinism checks, execute the 18-configuration notebook comparison, and record structural metrics and a provisional baseline. |
| 2. Embeddings | Vertex AI Embeddings, dimensions, semantic similarity, query and document inputs, compatibility, and model versioning. | Record model and dimension for each run and reproduce similarity comparisons with compatible query and document vectors. |
| 3. Vector search | PostgreSQL, pgvector, persistence, similarity search, metadata filtering, indexing, and retrieval quality. | Persist vectors, reproduce ranked and filtered queries, and compare index strategies on labeled queries. |
| 4. RAG pipeline | Query embedding, vector retrieval, context assembly, Gemini integration, source attribution, and retrieval evaluation. | Answer sample queries with references to retrieved chunks and report retrieval metrics for a labeled set. |
| 5. AI agents | Google ADK configuration, tools, function calling, RAG as a tool, and agent execution. | Run an agent that invokes the RAG tool and returns attributable results for representative prompts. |
| 6. Memory and sessions | Conversation history, user sessions, persistent state, and long-term memory experiments. | Restore session state across runs, verify isolation between users, and document the retention and retrieval behavior observed in the memory experiment. |
| 7. Application | FastAPI, React, TypeScript, Vite, chat, document upload, and authentication; Firebase is the current candidate. | An authenticated user can upload and query authorized documents; negative tests prevent cross-user retrieval and source access. |
| 8. Google Cloud integration | Cloud Storage, Pub/Sub, Cloud Run, and asynchronous ingestion. | Demonstrate ingestion from upload to searchable chunks in the approved environment and verify duplicate deliveries do not create duplicate records. |
| 9. Production engineering | Observability, evaluation, security, error handling, retries, and cost optimization. | Trace a request across components, measure quality and cost on representative traffic, and verify recovery and access controls. |
| 10. Multiagent systems | Conditional ADK coordination experiment; consider LangGraph only for a concrete comparative question. | Compare a repeatable multiagent workflow with the single-agent baseline on quality, latency, and cost; retain it only if justified. |

## Decision gates

- Before Phase 2: choose the actual embedding model, input modes, token limits, dimensions, and immutable configuration identifier. Record re-embedding behavior and any cloud-cost approval.
- Before Phase 3: settle local PostgreSQL/pgvector setup and index schema, including document identity, processing configuration, embedding configuration, and metadata filters. Establish exact-search baseline before approximate indexing; do not force ANN indexes on a tiny corpus.
- Before Phase 4: define labeled queries, retrieval and grounded-answer metrics, context budget, and source-reference format.
- Before Phases 5–6: define tool boundaries, failure behavior, session ownership, retention, deletion, and memory evaluation. Session history and retrieved long-term memory are different concerns.
- Before Phase 7: record authentication choice, document ownership/access rules, and negative access tests.
- Before Phase 8: justify asynchronous ingestion by its learning or runtime requirement, select database hosting/connectivity, and specify job state, idempotency, retries, failure visibility, and budgets. Pub/Sub and extra handlers are not automatic prerequisites for Cloud Run.
