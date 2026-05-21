# UGH Ecosystem

Deterministic semantic audit for AI-generated outputs across modalities.

## Overview

The UGH Ecosystem is a family of independent component repositories that
share a single design pattern: compare a **declared intent** against an
**observed output**, compute a **semantic distance**, return a stable
**verdict**, and emit a **repair surface** that can be acted on without
re-running the generator.

The shared problem is *intent drift* — the gap between what a producer
(a human author, an LLM, or a generation pipeline) said it would produce
and what was actually produced. Each component applies the same pattern
to one modality, with its own domain-specific extractors, constraints,
and repair vocabulary.

The ecosystem is readable as either a research program (a concrete
operationalisation of a theoretical framework) or as a practical family
of OSS audit tools. Both readings are intended.

## Domains

| Domain | Repository | What it audits | Current status |
|---|---|---|---|
| Text | https://github.com/Yuu6798/ugh-audit-core | AI Q&A response semantic honesty (PoR / ΔE / grv metrics over a declared response intent) | Phase 8 shipped; HA63 (n=63) validation complete; deterministic CLI + REST API + MCP server all shipped |
| Code | https://github.com/Yuu6798/semantic-ci-code | Python PR intent drift between a declared `target.yaml` and the observed code state extracted from the change | Active development; 8 CLI subcommands; discipline tests CI-enforced; no tagged release yet |
| Music | https://github.com/Yuu6798/ugh-prompt-engine | Audio quality drift between a Target SVP and an observed RPE bundle extracted from a WAV/MP3 | Proof of concept; 2 tags; ~217 commits; explicitly self-acknowledged as not yet production-validated (validation dataset pending) |
| Image + Video | https://github.com/Yuu6798/svp-video-pipeline | Image and short-video generation drift via an SVP 5-layer schema audit with a structured repair loop | Experimental; ~121 commits; ~$1.60 per standard run; catches C-group risks such as reversed hands, thin linear objects, and soft-body deformation |

Each repository has its own release cadence, contribution policy, and
runtime requirements. None of them depends on the others at runtime; the
shared surface is the design pattern, not a shared library.

## Core design pattern

All four components implement the same five-step pipeline:

```
Declared intent  →  Observed state  →  ΔE (semantic distance)  →  Verdict  →  Repair
```

The vocabulary differs per domain but the structural roles match. For
example, what `target.yaml` is to the code domain, `core_propositions`
is to the text domain; what a `RepairPlan` is to the code domain, a set
of ΔE-driven repair opcodes is to the text domain, and a
`target_svp.proposed.json` is to the image+video domain.

A full cross-domain vocabulary mapping — declared intent, observed state,
distance metric, verdict surface, repair surface — is deferred to a
future `docs/vocabulary.md`. The intent of this section is only to make
the cross-domain isomorphism legible, not to teach any one component's
schema in detail; each component repository documents its own surface.

## Architectural strata

The ecosystem deliberately separates two strata:

- **Audit layer — always deterministic.** Across all four domains, the
  drift detection, verdict, and repair surface are computed by
  deterministic code. No LLM call, no API key, no network is required to
  reproduce a verdict from a given (declared intent, observed state) pair.
- **Generation layer — varies by domain.** Text, Code, and Music
  components do not generate output; they only audit. The Image + Video
  component additionally includes a generation layer that uses an LLM
  planner and image/video backends, and applies the deterministic audit
  layer on top of that generation layer.

This stratification is by design, not an oversight: it is the reason a
component repository can simultaneously enforce a strict "no LLM in the
audit path" scope guard and acceptably use an LLM in its generation
pipeline. The invariant the ecosystem preserves is that the audit
verdict, once issued, can be reproduced from the recorded inputs alone,
without re-invoking any non-deterministic component. A fuller treatment
of the strata, the invariants each side preserves, and how the two
layers compose is deferred to a future `docs/strata.md`.

## Theory foundation

The ecosystem is grounded in the UGH (Unconscious Gravity Hypothesis)
framework, a theoretical / design framework for modelling semantic
gravity and drift in generated outputs. The four component repositories
are concrete operationalisations of that framework, one per modality.

Detailed theoretical exposition — the formal definitions, the
relationship to the per-domain distance metrics, and the lineage of
the design pattern — is deferred to a future `docs/theory.md` and is not
restated here.

## Status

This is the umbrella repository for ecosystem-level information. It is
intentionally docs-only on day 1: a discovery point and citation target,
not a runtime artefact.

Component repositories are independent and have their own release
cadences, issue trackers, and contribution policies. Cross-references
between repositories (shared schema notes, vocabulary mappings, ecosystem
roadmap) are being added incrementally and will land here as separate
documents under `docs/` when they are ready.

## License

MIT — see [LICENSE](./LICENSE).
