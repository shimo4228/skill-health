# skill-health

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/skill-health)

An [Agent Skill](https://agentskills.io/specification) that runs a **deterministic structural lint over a skill library**: it scans every `SKILL.md` for *missing-artifact* debt — a reference to a `scripts/` module, a bash script, an agent, or a sibling skill that no longer exists on disk, silently dangling after a rename or delete. The check is decidable by a filesystem `exists()` call, so it is owned by code at 100% accuracy. Everything that needs *judgment* — is a skill good, risky, or validated — is delegated, never re-scored here.

## Install

### Claude Code

```bash
# Copy into your global skills directory
cp -r skills/skill-health ~/.claude/skills/skill-health
```

### SkillsMP

```bash
/skills add shimo4228/skill-health
```

## How It Works

1. **Inventory** — enumerate `~/.claude/skills/*/SKILL.md` (plus any project-local skills); state up front how many will be scanned
2. **Compatibility scan** — extract every explicit local reference (`python -m scripts.X`, `bash …/x.sh`, `~/.claude/agents/X.md`, Markdown links to local files / sibling skills) and report those whose target does not exist
3. **Federate the rest** — surface the *existing* signals for Utility, Risk, and Validation by labelling each value's source; never recompute them here
4. **Report & ledger** — render a `skill × dimension` table and persist the JSON so the next run can diff

The scanner is **conservative by design**: it skips template placeholders (`<your-repo>/x.sh`), illustrative example links (`[](url)`), and `--directory`-overridden commands, because a false "missing artifact" is worse than a missed one. The exit code is the code-owned gate: `0` clean, `1` dangling references found, `2` scan root missing.

## Quick Example

```bash
uv run --directory ~/.claude/skills/skill-health \
  python -m scripts.scan_refs ~/.claude/skills
# add --json for machine-readable output
```

```
skill-health: structural reference scan (42 skill(s))

1 dangling reference(s) — 'missing artifacts' debt:

  [run_module] my-skill L61: scripts.old_name → /…/my-skill/scripts/old_name.py (missing)

Structural 'missing artifacts' check only — the semantic debt patterns
(over-specialized, trigger↔body drift), the risk dimension, and the validation
dimension are delegated to skill-stocktake / security-scan / skill-comply.
```

It does not auto-fix: a dangling reference may mean the artifact was *deleted* (remove the reference), *renamed* (repoint it), or *vendored from a repo* (a portability issue). The repair is a human judgment — the skill surfaces the fact, the user decides.

## When It Triggers

- "scan skills for debt" / "check for dangling or broken references in my skills"
- "skill health check" / "do the referenced scripts and agents still exist?"
- `/skill-health`

Not for holistic skill *quality* verdicts (that is [skill-stocktake](https://github.com/shimo4228/skill-stocktake)), config GC across hooks/permissions/MCP (that is [config-gc](https://github.com/shimo4228/config-gc)), or security scanning (that is [security-scan](https://github.com/shimo4228/security-scan)).

## The Four Dimensions

[SkillOps](https://arxiv.org/abs/2605.13716) frames skill-library health as four dimensions. `skill-health` owns the one structural dimension as deterministic code and federates the other three to the skill that already owns them:

| Dimension (SkillOps) | Owner | `skill-health` does |
|----------------------|-------|---------------------|
| **Compatibility** — references resolve | **`skill-health`** | the deterministic scan (100% accuracy) |
| **Utility** — frequency, value | `skill-stocktake` | read its signal; flag over-specialized |
| **Risk** — security, side effects | `security-scan` | delegate; prompt to run if stale |
| **Validation** — tests / consistency | `skill-comply` / `skill-creator` | note which skills lack a validator |

This is the **code / structural layer of the AKC Curate phase**, paired with [skill-stocktake](https://github.com/shimo4228/skill-stocktake) as the semantic / quality layer — a concrete instance of the guard pattern in [AKC ADR-0008 "Code-LLM Layering"](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0008-code-and-llm-collaboration.md): code owns the structural, decidable property; the LLM owns meaning.

## Syncing from the harness

The canonical copy of this skill lives in the author's live Claude Code harness. This repository is a one-way publication mirror:

```bash
scripts/sync-from-local.sh --dry-run   # report differences only
scripts/sync-from-local.sh             # apply to working tree (never commits)
```

## References

`skill-health` operationalizes the structural slice of a research framing on agent-skill technical debt. The papers it originates from:

- **SkillOps** — Pu, C., Song, Y., & Zhao, Y. (2026). *SkillOps: Aligning Software Engineering for Autonomous Agents.* arXiv:[2605.13716](https://arxiv.org/abs/2605.13716). **The origin paper.** skill-health's "skill technical debt" framing and the four-dimension health rubric (Compatibility / Utility / Risk / Validation) come from here; this skill owns the **Compatibility** (missing-artifact) dimension as deterministic code and federates the other three.
- **SoK: Agentic Skills** — Jiang, Y., et al. (2026). *A Comprehensive Survey of Agentic Skills: From Discovery to Execution.* arXiv:[2602.20867](https://arxiv.org/abs/2602.20867). The field map of the skill lifecycle that locates where curation sits.
- **How Well Do Agentic Skills Work in the Wild** — Liu, Y., et al. (2026). arXiv:[2604.04323](https://arxiv.org/abs/2604.04323). The empirical counterpart: skill benefits degrade toward the no-skill baseline as an uncurated library grows — independent support for curation as the load-bearing act.

## About this skill

This skill is the Curate-phase code layer of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) research line — a Zenodo-citable six-phase bidirectional growth loop ([DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)) for sustaining intent alignment between an AI agent and its operator over time. It instantiates the guard pattern of [AKC ADR-0008](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0008-code-and-llm-collaboration.md) (Code-LLM Layering) and is named the Curate code-layer in [AKC ADR-0019](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0019-cycle-structure-is-provisional.md) (the cycle's structure is provisional — skill-health clears dangling-reference debt before skill-stocktake judges quality). AKC is one of three research lines by [@shimo4228](https://github.com/shimo4228), alongside [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) ([DOI 10.5281/zenodo.19212118](https://doi.org/10.5281/zenodo.19212118)) and [Agent Attribution Practice (AAP)](https://github.com/shimo4228/agent-attribution-practice) ([DOI 10.5281/zenodo.19652013](https://doi.org/10.5281/zenodo.19652013)).

## License

MIT
