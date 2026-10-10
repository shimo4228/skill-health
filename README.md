# skill-health

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/skill-health)

An [Agent Skill](https://agentskills.io/specification) for Claude Code that runs a **deterministic structural lint over a skill library**: it scans every `SKILL.md` for *missing-artifact* debt, meaning a reference to a `scripts/` module, a bash script, an agent, or a sibling skill that no longer exists on disk and silently dangles after a rename or delete. It is for people who keep their own skills under `~/.claude/skills/` and rename or delete things often.

The scan reads local files only, and the skill edits no skill: it reports each dangling reference and leaves the repair to you. The author's other work is listed under [More from the author](#more-from-the-author).

## Install

The scanner runs on its own, but the full `/skill-health` report reads its usage numbers through [skill-stocktake](https://github.com/shimo4228/skill-stocktake)'s script, and skill-stocktake runs skill-health's scanner first, so install the two together. Both run their scripts with [`uv`](https://docs.astral.sh/uv/) and Python 3.11 or later.

There are two routes. Cloning (first block) installs the pair alone. The [akc-cycle](https://github.com/shimo4228/akc-cycle) Claude Code plugin (second block) installs both with the other skills of the Agent Knowledge Cycle (AKC: the author's six-phase, human-gated cycle that turns a coding agent's repeated experience into skills and rules).

```bash
git clone https://github.com/shimo4228/skill-health
git clone https://github.com/shimo4228/skill-stocktake
mkdir -p ~/.claude/skills
cp -r skill-health/skills/skill-health skill-stocktake/skills/skill-stocktake ~/.claude/skills/
```

With the clone install, the skill calls its scripts at `~/.claude/skills/skill-health` and `~/.claude/skills/skill-stocktake`, so keep the folders at those paths. Run it by typing `/skill-health`. Claude does not start it on its own: the skill sets `disable-model-invocation: true`, so it stays out of every session's context until you call it.

In the plugin the skill is called `/akc-cycle:skill-health`. This repository is synced one way from the same source, so between syncs it can trail the plugin.

```
/plugin marketplace add shimo4228/akc-cycle
/plugin install akc-cycle@akc-cycle
```

## Quick Example

The command and output below assume the clone install above. With the plugin, run `/akc-cycle:skill-health` instead.

```bash
uv run --directory ~/.claude/skills/skill-health \
  python -m scripts.scan_refs ~/.claude/skills
# add --json for machine-readable output
```

```
skill-health: structural reference scan (42 skill(s))

1 dangling reference(s) — 'missing artifacts' debt:

  [run_module] my-skill L61: scripts.old_name → /…/my-skill/scripts/old_name.py (missing)

4 unresolved name reference(s) — NOT a defect claim. Each names a skill/command that
is not under the skills root; it may still exist as a CLI builtin, bundled skill,
plugin command, or project-scoped skill. Decide per item:

  [skill_name] skill-health L52: `/claude-security` → 'claude-security' (unresolved)
  [skill_name] skill-health L118: `/skill-doctor` → 'skill-doctor' (unresolved)
  [skill_name] skill-health L136: `/claude-security` → 'claude-security' (unresolved)
  [skill_name] skill-stocktake L325: `/skill-doctor` → 'skill-doctor' (unresolved)

[closing note on the dimensions the scan leaves to other owners omitted]
```

The four unresolved names come from the skill-health and skill-stocktake `SKILL.md` files themselves, so the first run after the clone install lists them. They are expected: `/claude-security` is the security plugin command in the table below and `/skill-doctor` is a Claude Code command that reports how much each skill's listing adds to every turn ([Claude Code docs](https://code.claude.com/docs/en/skills#find-unused-skills), as of 2026-10-10), and neither is a folder under the skills root. Unresolved names do not change the exit code; a dangling reference does.

It does not auto-fix: a dangling reference may mean the artifact was *deleted* (remove the reference), *renamed* (repoint it), or *vendored* (authored in another repo and copied in, a portability issue). The repair is a human judgment: the skill surfaces the fact, the user decides.

## The Four Dimensions

[SkillOps](https://arxiv.org/abs/2605.13716) frames skill-library health as four dimensions. `skill-health` owns the one structural dimension as deterministic code. For the other three it reads the signals of whatever already owns them rather than recomputing them, and adds two LLM judgments of its own in the Utility and Validation rows:

| Dimension (SkillOps) | Owner | `skill-health` does |
|----------------------|-------|---------------------|
| **Compatibility**: references resolve | **`skill-health`** | the deterministic scan |
| **Utility**: frequency, value | `skill-stocktake` | read its usage signal; judge (LLM) whether a rarely used skill is over-specialized or just new |
| **Risk**: security, side effects | Anthropic's [`/claude-security`](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-security) plugin | delegate; prompt to run if stale |
| **Validation**: tests / consistency | `skill-comply` / `skill-creator` | note which skills lack a validator; judge (LLM) trigger↔body consistency where it is in doubt |

Why code owns the one dimension it does is in [AKC ADR-0008 "Code-LLM Layering"](https://github.com/shimo4228/agent-knowledge-cycle/blob/main/docs/adr/0008-code-and-llm-collaboration.md) and the folded section at the end.

## How It Works

1. **Inventory**: enumerate `~/.claude/skills/*/SKILL.md` (plus any project-local skills); state up front how many will be scanned
2. **Compatibility scan**: extract every explicit local reference (`python -m scripts.X`, `bash …/x.sh`, `~/.claude/agents/X.md`, Markdown links to local files / sibling skills) and report those whose target does not exist. The same run lists two things that are not defects: skill names written without a path that match no file (they may be CLI builtins or plugin commands), and skills whose folder is a symlink into someone else's tree, where a fix would be overwritten on their next upgrade
3. **Read the other three**: surface the *existing* signals for Utility, Risk, and Validation by labelling each value's source, and list `SKILL.md` bodies over 500 lines; never recompute them here
4. **Report & ledger**: render a `skill × dimension` table and save the scan as JSON to `~/.claude/skills/skill-health/results.json` so the next run can diff

The scanner is **conservative by design**: it skips template placeholders (`<your-repo>/x.sh`), illustrative example links (`[](url)`), and `--directory`-overridden commands, because a false "missing artifact" is worse than a missed one. The exit code is the code-owned gate: `0` clean, `1` dangling references found, `2` scan root missing.

## When to Run It

- When you want to know whether the scripts, agents and files your skills name still exist (dangling or broken references)
- After renaming or deleting a skill, script or agent

Not for overall skill *quality* verdicts (that is [skill-stocktake](https://github.com/shimo4228/skill-stocktake)) or security scanning (that is the `/claude-security` plugin).

## References

The papers skill-health draws on:

- **SkillOps**: Pu, C., Song, Y., & Zhao, Y. (2026). *SkillOps: Aligning Software Engineering for Autonomous Agents.* arXiv:[2605.13716](https://arxiv.org/abs/2605.13716). The origin paper: the "skill technical debt" framing and the four-dimension rubric come from here.
- **SoK: Agentic Skills**: Jiang, Y., et al. (2026). *A Comprehensive Survey of Agentic Skills: From Discovery to Execution.* arXiv:[2602.20867](https://arxiv.org/abs/2602.20867). A survey of the skill lifecycle.
- **How Well Do Agentic Skills Work in the Wild**: Liu, Y., et al. (2026). arXiv:[2604.04323](https://arxiv.org/abs/2604.04323). An empirical study of skills in use.

## More from the author

- **[akc-cycle](https://github.com/shimo4228/akc-cycle)**: installs this skill together with the rest of the cycle as one Claude Code plugin.
- **[Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle)**: the reasoning behind each phase of the cycle, including Curate, the phase that audits saved skills for structural and semantic debt and that skill-health belongs to, recorded as dated design decisions.
- **[skill-stocktake](https://github.com/shimo4228/skill-stocktake)**: the judgment layer that runs this scan first, then audits each skill for staleness, conflicts and redundancy with a verdict per skill; install it alongside.
- **[generation-audit](https://github.com/shimo4228/generation-audit)**: re-checks your own rules, skills and agents when a new Claude model takes over a role, and uses this scanner for the structural part.
- **[skill-comply](https://github.com/shimo4228/skill-comply)**: the Validation side of the table; it runs generated scenarios and reports how often agents actually follow a skill.
- **[shimo4228](https://github.com/shimo4228/shimo4228)**: the author's hub, with AKC next to the other long-running practice lines and their DOIs.

## License

MIT

<details>
<summary>For tools and AI assistants</summary>

skill-health is an Agent Skill for Claude Code that scans a skill library for references to scripts, agents, files or sibling skills that no longer exist on disk, for people who keep their own skills under `~/.claude/skills/` and rename or delete things often enough that references dangle silently. It reports facts and never edits a skill.

It exists because this one kind of skill debt is structural: whether a referenced path exists is answered exactly by `Path(target).exists()`, so it belongs in deterministic code rather than in an LLM's per-item attention, where an audit can assign Keep to a skill whose scripts are gone. In the author's design (AKC ADR-0008, Code-LLM Layering), code owns what is decidable and the LLM owns meaning; AKC ADR-0019 names skill-health the Curate code layer, which clears dangling-reference debt before skill-stocktake judges quality. Utility, risk and validation are read from their owners (skill-stocktake's usage script, the `/claude-security` plugin, skill-comply and skill-creator) and never recomputed; the skill's procedure adds only two LLM judgments, whether a rarely used skill is over-specialized and, where in doubt, whether a skill's trigger matches its body. The framing comes from SkillOps (arXiv:2605.13716), which names the four dimensions; SoK: Agentic Skills (arXiv:2602.20867) maps where curation sits in the skill lifecycle, and Liu et al. (arXiv:2604.04323) report that skill benefits degrade toward the no-skill baseline as an uncurated library grows, which supports curation as the load-bearing act.

Canonical facts: MIT license; a `SKILL.md` plus two Python scripts, `scripts/scan_refs.py` (the reference scanner; `--json` for machine output, `--external-urls` to list the external URLs the skills name) and `scripts/url_liveness.py` (classifies those URLs as `live`, `dead`, `blocked` or `skip`; skill-stocktake runs it once, serially, during its audit), Python 3.11 or later run through `uv` with the bundled `uv.lock`, tests under `tests/`; maintained by one author (@shimo4228). Status: active, synced one way from the author's Claude Code harness by `scripts/sync-from-local.sh` (`--dry-run` reports differences only; it never commits), and also shipped in the akc-cycle plugin as `/akc-cycle:skill-health`, so this repository can trail the plugin between syncs. Requirements: Claude Code; `uv`; the skill-stocktake skill installed at `~/.claude/skills/skill-stocktake` for the Utility column; no paid key and no network for the scan itself. It runs only when called as `/skill-health` (`disable-model-invocation: true`) and writes its last scan to `~/.claude/skills/skill-health/results.json`.

Example: `uv run --directory ~/.claude/skills/skill-health python -m scripts.scan_refs ~/.claude/skills` prints the number of skills scanned, then each dangling reference as `[ref_type] skill Lline: raw → resolved path (missing)`, where `ref_type` is one of `run_module`, `bash_script`, `agent` or `md_link`, and exits `0` when clean, `1` when dangling references are found, and `2` when the scan root is missing. Unresolved names and symlinked external skills are listed separately and do not change the exit code. The skill then renders `Skill | Compatibility | Utility | Risk | Validation`, where only Compatibility (the dangling count) is a hard number.

Links: [skills/skill-health/SKILL.md](skills/skill-health/SKILL.md) is the skill itself; [llms.txt](llms.txt) and [llms-full.txt](llms-full.txt) are the machine-readable summary and reference. The skill implements the code layer of the Curate phase of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle), concept DOI [10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726); cite AKC by that DOI. The installable form of the whole cycle is [akc-cycle](https://github.com/shimo4228/akc-cycle).

</details>
