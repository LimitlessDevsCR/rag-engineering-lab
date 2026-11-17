# P1-001: Document extraction and chunking

Status: Proposed — awaiting explicit user approval. Not ready for development.
Date: 2026-11-17

## Objective and architectural problem

Produce reproducible, attributable text chunks from local PDF and TXT files. Learn how extraction, normalization, boundaries, size, and overlap change the material that a later retrieval system will index. This phase measures structural behavior; it cannot establish retrieval quality without embeddings and labeled retrieval queries.

## Scope and approach

Use synchronous Python functions and standard-library dataclasses in the existing package. Use `pypdf` for digitally generated PDFs and standard-library UTF-8 decoding for TXT. Keep PDF pages as separate extraction units and treat a TXT file as one unit. Chunk within each unit; never cross PDF page boundaries in this phase. This simplifies exact source attribution at the cost of context across page breaks.

Data flow: local file bytes → content hash and source metadata → raw extraction units → normalized units → chunks → notebook comparisons and local JSONL output.

Preserve raw extracted text alongside normalized text. Offsets address normalized text, not PDF byte positions or raw text positions. No coordinate mapping back to raw text is required.

## Components and deliverables

- Reusable modules under `src/rag_engineering_lab/` for records, extraction, normalization, chunking, and JSONL serialization. Module names are implementation choices; no plugin registry, framework, abstract provider layer, or service is required.
- `notebooks/01_document_extraction_chunking.ipynb` imports reusable functions and contains the experiment, result tables, boundary examples, limitations, and recommendation.
- Synthetic fixtures and a provenance manifest under `samples/phase01/`.
- Meaningful automated checks under `tests/`.
- `docs/architecture/phase-01-results.md` records measured results, reproduction commands, environment versions, and the selected baseline after implementation.

## Inputs and outputs

Input: a local PDF or UTF-8 TXT path, an explicit portable source path relative to the input root, and a chunk configuration. Local absolute paths must not appear in persisted metadata. No directory watcher or upload API.

Output: extraction records, structured diagnostics, chunk records, and UTF-8 JSONL with one chunk per line. Return records directly so notebook use does not require serialization. Keep reproducible output separate from timings and run-specific timestamps.

### Record contracts

| Record | Required fields and semantics |
| --- | --- |
| Document | `document_id`: SHA-256 hex of original bytes; `source_path`: relative POSIX path; `file_name`; `media_type`; `byte_size`; `page_count`: PDF count or null for TXT; `title` and `author`: nullable strings from PDF metadata, null for TXT. |
| Extraction unit | `document_id`; `unit_index`: zero-based; `page_number`: one-based for PDF, null for TXT; `raw_text`; `normalized_text`; `normalization_version`. Retain units with empty text. |
| Chunk | `chunk_id`; `document_id`; `source_path`; `file_name`; `media_type`; `page_number`; `unit_index`; `chunk_index`: zero-based within the unit; `text`; `start_char`; `end_char`; `strategy`; `chunk_size`; `chunk_overlap`; `normalization_version`; `chunker_version`. |
| Diagnostic | `source_path`; `page_number` when applicable; stable `code`; human-readable `message`. |

Offsets use Python string indices with an inclusive start and exclusive end. Every chunk must satisfy `text == normalized_text[start_char:end_char]`. Every chunk joins to its document metadata through `document_id`; source fields are also carried directly for attribution. Identical bytes at different paths share a document ID but retain distinct source records. This phase does not implement deduplication or a document catalog.

Generate chunk IDs using SHA-256 over a canonical, sorted-key JSON representation of document ID, unit index, normalization version, chunker version, strategy, size, overlap, and offsets. Content changes or processing configuration changes must not silently reuse incompatible IDs. Source-path changes alone do not change chunk IDs. No timestamps or random IDs enter this calculation.

## Technical requirements

### Extraction and normalization

1. Read bytes once for content identity. Decode TXT with strict `utf-8-sig` to accept UTF-8 BOM; report invalid encoding without replacement characters or encoding guessing.
2. Extract PDF text page by page in source order. Preserve page numbers even when preceding pages yield no text. Capture supported title and author metadata without requiring either.
3. Normalization v1 removes a leading Unicode BOM if present and converts CRLF/CR to LF. Preserve case, accents, punctuation, paragraph breaks, indentation, and internal spaces. Do not remove headers, repair hyphenation, collapse whitespace, or infer reading order.
4. Whitespace-only units produce no chunks and a `NO_EXTRACTABLE_TEXT` diagnostic. This does not prove that the page is scanned. Keep usable pages from mixed PDFs.
5. Report missing/unreadable files, unsupported extensions, invalid TXT encoding, malformed PDFs, and encrypted PDFs with explicit errors. Reject encrypted PDFs in this phase, including those with empty passwords. Never represent extraction failure as a successful empty document. A batch in the notebook may capture errors per file and continue while displaying failures.
6. Document the limits of PDF extraction: no OCR, layout reconstruction, table parsing, or reliable reading-order guarantees for arbitrary PDFs.

### Chunking strategies

Implement two explicit strategies using character counts, not token counts:

- `fixed`: slice contiguous windows of at most `chunk_size`, advancing to `end_char - chunk_overlap` until the end of the unit.
- `paragraph`: use the same window limit, but prefer the last boundary immediately after a blank-line separator within the candidate window. Accept that boundary only if `end_char - start_char > chunk_overlap`; otherwise use the hard window end. Start the next window at `end_char - chunk_overlap`. Preserve the separator within the preceding chunk. A long paragraph therefore falls back to fixed slicing.

Both strategies require integer `chunk_size > 0` and `0 <= chunk_overlap < chunk_size`; invalid inputs raise an explicit error. Emit no zero-length chunks. Stop when the chosen end reaches unit length. Never trim chunk text or search for substring offsets. Windows must cover every character of each non-whitespace unit, make strictly positive forward progress, and obey the size limit. Exact overlap is required between consecutive windows, including paragraph-aware windows. Character windows can split words; record representative examples rather than concealing this limitation.

### Experiment design

- Test both strategies at sizes 400, 800, and 1,200 characters and overlaps 0, 80, and 160 characters: 18 configurations on the same corpus.
- Use at least three TXT fixtures covering short text, multiple paragraphs, and a paragraph longer than 1,200 characters. Include accents, repeated text, BOM, and mixed line endings across fixtures.
- Include a digitally generated PDF with at least three pages, distinguishable page markers, a blank page between text pages, and title/author metadata. Include an image-only PDF to demonstrate the no-text diagnostic.
- Add focused fixtures for empty/whitespace TXT, invalid UTF-8, malformed PDF, and encrypted PDF. They are validation cases, not part of aggregate chunk-quality comparisons.
- Record fixture provenance and expected markers/page locations in a manifest. Use only synthetic content. Commit reproducible generation instructions or a fixture-generation helper with declared test-only dependencies; commit the small fixtures so extraction tests do not require regeneration.
- For each document/configuration report chunk count, min/median/max length, coverage, redundant-character ratio, and paragraph-boundary alignment. Define redundancy as `(sum(chunk lengths) - covered characters) / covered characters`; report not applicable for empty inputs. Define boundary alignment over non-final chunk ends that fall immediately after blank-line separators. Report corpus aggregates with explicit denominators.
- Show examples of paragraph breaks and long-paragraph fallback. Inspect at least six source passages across PDF and TXT and report whether each passage fits intact in any chunk, using fixture offsets. Passage containment is a structural proxy, not evidence of semantic relevance.
- Record elapsed extraction/chunking time and environment/package versions for reproducibility, without imposing machine-specific performance thresholds.
- Select a provisional configuration from measured boundary behavior and redundancy, document tradeoffs, and defer any claim of optimal RAG chunking until retrieval evaluation.

## Dependencies

Phase 0 scaffold only; no cloud account or credentials. Proposed additions on approval:

- Runtime: `pypdf`.
- Development: `pytest`, Jupyter tooling and `ipykernel`, plus notebook execution tooling if required for the validation command.
- Fixture generation only: `reportlab` if needed to generate synthetic PDFs. Keep it out of runtime dependencies.

Use the standard library for records, hashing, JSONL, statistics, and timing. No pandas, LangChain, tokenizer, embedding SDK, vector database, or API framework is needed. The Developer must resolve compatible versions against Python 3.12, declare dependency groups, and record the tested versions and exact setup commands. Do not invent version pins in this proposal.

## Acceptance criteria

1. A clean Python 3.12 environment can install declared dependencies and run the documented validation commands without credentials or network access after installation.
2. TXT normalized output matches the fixture expectation exactly. PDF fixture markers appear on the expected pages, metadata survives extraction, and the blank intermediate page does not shift attribution.
3. Every chunk retains source attribution and resolves to exactly the corresponding normalized slice. Tests exercise both strategies, overlaps, short inputs, repeated text, long paragraphs, empty units, invalid configurations, and Unicode.
4. For each non-whitespace unit and all 18 configurations, interval coverage is 100%, window size stays within the limit, overlap is exact, offsets advance, and repeated runs produce identical records and IDs.
5. Changing size, overlap, processing version, or source bytes produces the corresponding distinct identities. JSONL round trips preserve all chunk fields and Unicode.
6. Failure cases and no-text pages are visible through the stated diagnostics/errors; successful mixed-PDF pages still produce chunks. Image-only input never triggers an implicit OCR or cloud call.
7. Restarting the notebook kernel and executing all cells from repository-root instructions reproduces the complete 18-configuration comparison without hidden state. No absolute developer paths or sensitive notebook output are committed.
8. Results document provides measurements, a provisional baseline, known limitations, and a list of questions reserved for retrieval evaluation. The Developer updates current state and component status with evidence, without advancing to Phase 2.

## Validation approach

Run pytest against fixture-based extraction and invariant-based chunking tests, including parameterization across the comparison matrix. Execute the notebook non-interactively from a fresh kernel and inspect its tables and attribution examples. Review JSONL output for portability and reproducibility. Record exact commands and results; no embedding-based accuracy or performance claim is required.

## Explicit exclusions

OCR, DOCX/HTML, semantic or token-based chunking, layout-aware extraction, embedding generation, retrieval, LLM calls, cloud services, queues, databases, agents, APIs, UI, authentication, document authorization implementation, deployments, commits, and subsequent phases.

## Approval gate

The Architect has prepared this proposal, not authorized implementation. Only explicit user approval changes this task to Ready for development. Any material scope change requires an updated proposal. Approval of this task does not authorize Phase 2 or paid resources.
