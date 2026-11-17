# ADR-0001: Local, attributable document processing

Status: Proposed — awaiting user approval with P1-001.
Date: 2026-11-17

## Context

The repository is a Python scaffold without runtime dependencies. The first learning objective is understanding extraction and chunking, not operating a retrieval service. Source attribution and deterministic processing will be needed by later indexing and evaluation work.

## Decision

Propose synchronous local processing with pypdf for text-bearing PDFs, UTF-8 TXT decoding, explicit Python chunkers, immutable source-content hashes, and page-local normalized character offsets. Keep experiments in Jupyter and reusable behavior in the package. Use JSONL as an inspection/export artifact, not as a database. See [P1-001](../tasks/phase-01-document-extraction.md) for contracts and validation.

## Alternatives considered

- LangChain Text Splitters: useful as a later comparison, but not needed to learn and inspect these two algorithms.
- Layout-aware extractors or OCR: appropriate when a concrete corpus requires them; they broaden dependencies and evaluation beyond this phase.
- Cross-page chunks with span maps: better continuity, but require multiple page spans and more complex attribution. Revisit if retrieval evidence shows a material loss.
- Cloud document processing: adds credentials, cost, and operational concerns without a current learning requirement.

## Consequences

Local experiments need no paid resources. Records retain reproducible provenance and processing versions. Character counts do not enforce future model token limits. Page boundaries can interrupt context, and basic PDF extraction cannot guarantee layout fidelity. Retain these limits in results rather than adding speculative abstractions.

## References

- [pypdf extraction documentation](https://github.com/py-pdf/pypdf/blob/main/docs/user/extract-text.md): text extraction limitations and lack of OCR.
