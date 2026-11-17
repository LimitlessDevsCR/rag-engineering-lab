# Architecture overview

## Implemented

The repository contains documentation, Python package metadata, and an empty Python package. It has no application runtime or cloud integrations.

## Planned

These components describe the target direction, not completed integrations or blanket approval of service choices. The next proposed increment is [P1-001](../tasks/phase-01-document-extraction.md), supported by [proposed ADR-0001](../decisions/0001-local-document-processing.md). No Phase 1 component is implemented.

| Area | Components |
| --- | --- |
| Document ingestion | Proposed Phase 1: local synchronous pypdf/TXT extraction, page-local normalization, two explicit character chunkers, portable attribution, deterministic IDs, and JSONL export. Framework splitters remain optional future comparisons. |
| Embeddings and storage | Vertex AI Embeddings, PostgreSQL with pgvector, and metadata filtering. Track embedding model, version, and dimensionality alongside stored vectors. |
| Retrieval and generation | Embed the query, retrieve relevant chunks, assemble context, call Gemini through Vertex AI, and return source attribution. Evaluate retrieval with repeatable fixtures. |
| Agents and state | Google ADK for tools, function calling, session state, and access to the RAG pipeline as a tool. |
| Application | FastAPI backend and React/TypeScript with Vite. Firebase Authentication is a candidate; document-level authorization is a separate required boundary before multiuser use. |
| Google Cloud | Target candidates: Cloud Storage and Cloud Run. Pub/Sub is conditional on the asynchronous ingestion experiment. No additional function service without a distinct responsibility. Database hosting and connectivity remain undecided. |
| Engineering practices | pytest for automated tests, plus evaluation, observability, security controls, retries, and cost measurement. |

The ingestion path takes documents through extraction, normalization, and chunking, then embeds the chunks and persists their vectors and source metadata. The query path embeds a query using a compatible model, retrieves relevant chunks, and uses them to build the context and source references for a Gemini response. An ADK agent will call the same retrieval and generation workflow through a tool.

## Boundaries and unresolved decisions

Ingestion and querying are separate workflows in one Python package initially, not separate microservices. Notebooks invoke package functions and do not become production dependencies. The agent consumes the RAG workflow; core retrieval must not depend on agent execution or session state.

Phase 1 proposes content-addressed documents and chunks with processing versions. Phase 2 adds an explicit embedding configuration; Phase 3 must persist it alongside vectors and prevent incompatible queries. Reprocessing and re-embedding are distinct operations. Page-local offsets support attribution to extracted text; they do not represent visual PDF coordinates.

No database schema, embedding model, authentication provider, production topology, or memory backend is implemented. Resolve each at its roadmap gate. Maintain an exact-search baseline before adding approximate indexes, introduce diagnostic/quality checks in each phase, and define authorization before exposing multiple users' documents. Retrieved text must remain untrusted when used as generation context or tool input.

## Experimental

Jupyter notebooks will isolate experiments from application code. Planned experiments cover chunking, embedding compatibility, retrieval quality, long-term memory, and conditional multiagent coordination with Google ADK. LangGraph is optional and requires a specific comparative question. The roadmap defines when each experiment starts.
