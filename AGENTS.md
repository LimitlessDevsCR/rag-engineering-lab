# RAG Engineering Lab

## Start of each session

Read [current state](docs/current-state.md), the [roadmap](docs/roadmap.md), the [architecture overview](docs/architecture/overview.md), and any applicable records in `docs/decisions/`. Use these checked-in files as the shared project context across conversations.

## Collaboration

### Architect conversation

Define architecture, evaluate alternatives, establish phases and acceptance criteria, review decisions, identify risks, and maintain the roadmap and architecture records. Application code is the Developer's responsibility.

### Developer conversation

Implement approved tasks, modify source files, install dependencies needed by the approved phase, run tests, validate functionality, report results, and update technical documentation. Obtain approval before introducing major architectural changes or starting Phase 1.

At the end of each task, update `docs/current-state.md` with completed work, known issues, and the next recommended task. Record accepted architectural decisions in `docs/decisions/` and update the architecture overview when a component's status changes.

## Development rules

- Build one approved phase at a time with simple, explicit implementations. Add abstractions when a concrete use case requires them.
- Use open-source libraries when they support the learning objective. Introduce dependencies and cloud integrations when the active phase requires them.
- Keep reusable code under `src/rag_engineering_lab/` and experiments under `notebooks/`. Add automated checks under `tests/` as functionality is implemented.
- Document architectural decisions and the tradeoffs that drove them. Keep implemented, planned, and experimental components explicit in the documentation.
- Keep all repository content suitable for public distribution. Use synthetic samples or public data with terms that permit the intended use and redistribution. Exclude proprietary code, confidential assets, credentials, and secrets.
- Obtain explicit approval before provisioning paid resources. Prefer local experiments where they meet the learning objective.
