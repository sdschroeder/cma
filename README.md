# Cognitive Mesh Architecture (CMA)

Documentation and reference for the Cognitive Mesh Architecture and related systems: Ultra-Intelligent Content (UIC), StoryCycle Genie, and Brightsy implementation.

---

## Where to start

| If you want… | Start here |
|---------------|------------|
| **What CMA is and how it maps to Brightsy/Storycycle** | [docs/spec/cma-authoritative-reference.md](docs/spec/cma-authoritative-reference.md) |
| **Technical definition (source of truth)** | [docs/spec/cma-technical-spec.md](docs/spec/cma-technical-spec.md) |
| **Narrative and business case** | [docs/thought-leadership/cma-whitepaper.md](docs/thought-leadership/cma-whitepaper.md) |
| **Implementing CMA in Brightsy** | [docs/implementation/cma+brightsy-implementation-plan.md](docs/implementation/cma+brightsy-implementation-plan.md) |
| **StoryCycle Genie implementation** | [docs/implementation/storycycle-genie/storycycle-genie-v2-executable-vision.md](docs/implementation/storycycle-genie/storycycle-genie-v2-executable-vision.md) |

---

## Repository structure

| Path | Purpose |
|------|---------|
| [docs/spec/](docs/spec/) | **Source of truth:** authoritative reference, technical spec |
| [docs/implementation/](docs/implementation/) | Brightsy plan, StoryCycle Genie, Ultra-Intelligent Content (extended narrative) |
| [docs/thought-leadership/](docs/thought-leadership/) | Whitepaper, use cases (construction, healthcare, legal), IP guidelines, PDF |
| [docs/research/](docs/research/) | Industry and background research; alignment with CMA/UIC |
| [docs/archive/](docs/archive/) | Superseded StoryCycle Genie docs (design history) |
| [docs/](docs/) | Organization and consolidation notes |

---

## Key relationships

- **Technical spec** = single source of technical truth for CMA and UIC. Whitepaper and UIC docs provide narrative and examples.
- **Authoritative reference** = current state (Brightsy/Storycycle), gaps, and Path A vs Path B. Implementation plan points to it for goals and architecture.
- **StoryCycle Genie v2** = single implementation reference for the product; older token-optimization and complete-vision docs are in `docs/archive/`.
