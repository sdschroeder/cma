# CMA Project: Recommended Documentation Organization

**Purpose:** After reviewing docs and folder structure, this document suggests an optimal organization that reflects content purpose, reduces confusion, and scales as the repo grows.

> **Status:** This recommendation was **applied** (restructure completed). The current layout is `docs/spec/`, `docs/implementation/`, `docs/thought-leadership/`, `docs/research/`, `docs/archive/` as described below. Kept for rationale and for future changes. See [README](../README.md) for the current map.

---

## Current State Summary (before restructure)

| Location | Contents | Role |
|----------|----------|------|
| **Root** | `cma-authoritative-reference.md`, `cma-technical-spec.md`, `cma+brightsy-implementation-plan.md`, `cma.code-workspace` | Core CMA definition + Brightsy implementation bridge |
| **assets/** | Whitepaper, use cases (construction, healthcare, legal), IP guidelines, PDF | Thought leadership, use cases, compliance |
| **research/** | `intelligent-content-modeling-ai.md` | Industry/background research |
| **storycycle-genie/** | Vision, token optimization, record types, Brightsy review, strategic framework | Product/platform (Brightsy + StoryCycle) implementation |
| **ultra-intelligent-content/** | UIC overview, tech implementation, semantic schema framework | CMA evolution concept (UIC); extended narrative per technical spec |

**Issues identified:**
- **Root clutter:** Three major CMA docs at root; “assets” is vague and mixes whitepapers, use cases, and IP.
- **Unclear hierarchy:** Hard to tell at a glance what is source-of-truth vs narrative vs product vs research.
- **Link fragility:** Many internal links assume root-relative paths (e.g. `cma-technical-spec.md`, `cma-whitepaper.md`). Moving files without updating links will break references.
- **Single-file folder:** `research/` has one file; could be merged into a broader “background” or “research” area.

---

## Recommended Structure

Organize by **purpose and audience**: one clear “spec” area, one “implementation” area, one “thought leadership / use cases” area, and one “research” area. Keep the workspace and any repo-level config at root.

```
cma/
├── README.md                          # NEW: Entry point, map of the repo, link to spec
├── .gitignore
├── cma.code-workspace
│
├── docs/                              # All documentation (single top-level)
│   │
│   ├── spec/                          # Source of truth — canonical CMA/UIC definition
│   │   ├── cma-authoritative-reference.md
│   │   ├── cma-technical-spec.md
│   │   └── INDEX.md                   # Optional: "Start here for CMA definition"
│   │
│   ├── implementation/               # How we build it (Brightsy, StoryCycle, MCP)
│   │   ├── cma+brightsy-implementation-plan.md
│   │   ├── storycycle-genie/          # Product docs (move folder here)
│   │   │   ├── brightsy-review.md
│   │   │   ├── storycycle-genie-complete-vision-extension.md
│   │   │   ├── storycycle-genie-token-optimization-plan.md
│   │   │   ├── storycycle-genie-v2-executable-vision.md
│   │   │   ├── storycycle-labs-strategic-framework.md
│   │   │   └── storycycle-record-types-summary.md
│   │   └── ultra-intelligent-content/ # UIC extended narrative (per technical spec)
│   │       ├── ultra-intelligent-content.md
│   │       ├── ultra-intelligent-content-tech-implementation.md
│   │       └── ultra-intelligent-content-semantic-schema-framework.md
│   │
│   ├── thought-leadership/            # Use cases, whitepapers, IP (replace assets/)
│   │   ├── cma-whitepaper.md
│   │   ├── cma-ip-guidelines-thought-leadership.md
│   │   ├── cognitive-mesh-architecture-whitepaper-v1.pdf
│   │   └── use-cases/
│   │       ├── cma-construction-management-use-case.md
│   │       ├── cma-healthcare-use-case.md
│   │       └── cma-legal-use-case.md
│   │
│   └── research/                      # Industry / background research
│       └── intelligent-content-modeling-ai.md
│
└── (no other doc folders at root)
```

---

## Rationale

1. **Single `docs/` root**  
   All prose and specs live under `docs/`. Easier to onboard (“everything is in `docs/`”) and to add tooling (link checks, static site, etc.).

2. **`docs/spec/` = source of truth**  
   Authoritative reference and technical spec are the canonical CMA/UIC definition. Keeping them together in `spec/` makes that explicit and keeps root clean.

3. **`docs/implementation/`**  
   Groups “how we implement CMA”:
   - Brightsy implementation plan
   - StoryCycle Genie (vision, optimization, record types, Brightsy review)
   - UIC extended narrative and schema (as referenced by the technical spec)

   UIC stays next to implementation because the technical spec treats it as extended narrative/examples; the spec remains the single technical source of truth.

4. **`docs/thought-leadership/`**  
   Replaces the vague “assets” with a clear purpose: whitepapers, IP guidelines, and use cases. Optional `use-cases/` subfolder keeps use-case docs grouped.

5. **`docs/research/`**  
   Keeps industry/background (e.g. intelligent content modeling) in one place. You can add more research docs here without changing the rest of the tree.

6. **Root**  
   Only `README.md`, `.gitignore`, and `cma.code-workspace`. README points to `docs/spec/` and optionally to implementation and thought leadership.

---

## Link Updates Required

After moving files, internal links must be updated. Key patterns:

| From | Current link example | New path (example) |
|------|----------------------|--------------------|
| Whitepaper, IP guidelines (in thought-leadership) | `cma-technical-spec.md` | `../spec/cma-technical-spec.md` or `/docs/spec/cma-technical-spec.md` if using absolute-from-repo |
| Technical spec | `ultra-intelligent-content.md` | `../implementation/ultra-intelligent-content/ultra-intelligent-content.md` |
| Technical spec | `storycycle-genie-v2-executable-vision.md` | `../implementation/storycycle-genie/storycycle-genie-v2-executable-vision.md` |
| StoryCycle Genie docs | `cma-technical-spec.md` | `../../spec/cma-technical-spec.md` |
| StoryCycle Genie docs | `ultra-intelligent-content.md` | `../ultra-intelligent-content/ultra-intelligent-content.md` |
| Implementation plan | `cma-authoritative-reference.md` | `../spec/cma-authoritative-reference.md` |

Recommendation: do the move in one pass, then run a pass (e.g. grep or a script) for `](.*\.md)` and `](.*\.pdf)` to find and fix every relative link. Optionally add a CI check that validates internal doc links.

---

## Alternative: Minimal Change

If you prefer minimal disruption and fewer link edits:

- **Keep** `cma-authoritative-reference.md`, `cma-technical-spec.md`, and `cma+brightsy-implementation-plan.md` at **root** (so existing root-relative links keep working).
- **Rename** `assets/` → **`docs/thought-leadership/`** (or `docs/use-cases-and-whitepapers/`) and keep the same files; optionally add `thought-leadership/use-cases/` and move the three use-case files there.
- **Move** `research/` to **`docs/research/`** so all “docs” live under `docs/` except the three root spec/plan files.
- **Move** `storycycle-genie/` and `ultra-intelligent-content/` under **`docs/implementation/`** (or keep at root as `storycycle-genie/` and `ultra-intelligent-content/` and only rename `assets/` and `research/`).

Then add a short **README.md** at root that maps the repo (e.g. “Spec: cma-technical-spec.md, Authoritative ref: cma-authoritative-reference.md, Implementation: cma+brightsy-implementation-plan.md + storycycle-genie/ + ultra-intelligent-content/, Thought leadership: docs/thought-leadership/, Research: docs/research/”).

---

## Suggested README.md (for root)

Use this (or a variant) as the repo entry point:

```markdown
# Cognitive Mesh Architecture (CMA)

Documentation and reference for the Cognitive Mesh Architecture and related systems (Ultra-Intelligent Content, StoryCycle Genie, Brightsy).

## Where to start

- **What is CMA?** → [Authoritative Reference](docs/spec/cma-authoritative-reference.md)
- **Technical definition (source of truth)** → [Technical Specification](docs/spec/cma-technical-spec.md)
- **Implementing in Brightsy** → [CMA + Brightsy Implementation Plan](docs/implementation/cma+brightsy-implementation-plan.md)

## Repository structure

| Path | Purpose |
|------|---------|
| `docs/spec/` | Canonical CMA/UIC definition |
| `docs/implementation/` | Brightsy, StoryCycle Genie, UIC implementation narrative |
| `docs/thought-leadership/` | Whitepapers, use cases, IP guidelines |
| `docs/research/` | Industry and background research |
```

(If you keep the three main docs at root, point to them as `cma-authoritative-reference.md`, `cma-technical-spec.md`, `cma+brightsy-implementation-plan.md` and still list the rest under `docs/`.)

---

## Summary

- **Optimal organization:** One `docs/` tree with `spec/`, `implementation/`, `thought-leadership/`, and `research/`. Spec and implementation are clearly separated; “assets” becomes “thought-leadership” with an optional `use-cases/` subfolder.
- **Trade-off:** Moving files requires a one-time link update pass; keeping the three core docs at root reduces link changes but keeps a busier root.
- **Immediate win:** Add `README.md` at root and, if you keep current paths, only rename `assets/` to something like `docs/thought-leadership/` and move `research/` to `docs/research/` to clarify purpose without moving the spec files.
