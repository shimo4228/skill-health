# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] — 2026-06-30

Initial publication of skill-health as a standalone skill repository: a structural skill-library debt scan, the Curate code-layer of the Agent Knowledge Cycle.

### Added

- `skills/skill-health/SKILL.md` — the four-phase scan (Inventory / Compatibility / Federate / Report) for *missing-artifact* debt: a SKILL.md that references a `scripts/` module, bash script, agent, or sibling skill that no longer exists on disk. Owns the structural Compatibility dimension as deterministic code; delegates Utility / Risk / Validation to skill-stocktake / security-scan / skill-comply.
- `skills/skill-health/scripts/scan_refs.py` — the deterministic reference-existence scanner (conservative by design; exit code is the code-owned gate: `0` clean, `1` dangling, `2` scan root missing), with its pytest suite under `tests/`.
- `scripts/sync-from-local.sh` — one-way export from the live Claude Code harness; the harness copy is canonical, this repository is the publication mirror.

### Notes

- Origin: the four-dimension health rubric (Compatibility / Utility / Risk / Validation) and the "skill technical debt" framing come from SkillOps (Pu, Song & Zhao 2026, [arXiv:2605.13716](https://arxiv.org/abs/2605.13716)); skill-health owns the Compatibility (missing-artifact) dimension and federates the rest. See the [References](README.md#references) section.
- Cycle position recorded in [AKC ADR-0008](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0008-code-and-llm-collaboration.md) (Code-LLM Layering) and [AKC ADR-0019](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0019-cycle-structure-is-provisional.md) (the Curate code-layer before skill-stocktake).
