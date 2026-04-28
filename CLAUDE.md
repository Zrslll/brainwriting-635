# brainwriting-635 — project rules

This repository develops a Cursor / Claude **skill** for multi-agent brainwriting.
It is the **skill development project**, not the skill executing a user session.

## What this is

The `brainwriting-635` skill combines two structured thinking methods:

1. **Brainwriting 6-3-5** (Bernd Rohrbach, 1968) — divergence phase: six role-based
   subagents generate ideas in parallel over N rounds with “sheet” rotation
2. **Six Thinking Hats** (Edward de Bono, 1985) — convergence phase: five hat-based
   subagents validate each top concept in parallel

The main agent = Blue hat = orchestrator.

## Repository layout

```
brainwriting-635/
├── SKILL.md                  Main skill file (loaded by Cursor / Claude)
├── CLAUDE.md                 This file — maintainer rules (English)
├── CLAUDE.ru.md             Optional local Russian mirror (gitignored)
├── README.md                 Public GitHub documentation
├── .gitignore
├── config/                   Static configs (presets, hats, rotation)
├── prompts/                  Prompt templates and orchestrator instructions
├── schemas/                  JSON schemas for subagent outputs
├── references/               Deep docs for orchestrator to load
└── runs/                     Gitignored — run artifacts
```

## Language policy

**All skill files (`SKILL.md`, `prompts/`, `schemas/`, `references/`, configs) are in English.**

Reasons:

- Models follow English instructions more reliably
- Norm for open-source skills
- Fewer ambiguities in technical terms

**Optional Russian:** maintainers may keep a local `CLAUDE.ru.md` (same substance as this file). It is listed in `.gitignore` and is **not** committed. Final user-facing reports may be translated in a separate step when needed; requirements for **human-readable report structure** are in [Skill improvements: human-facing output](#skill-improvements-human-facing-output-and-report).

## Development principles

### YAGNI, strictly

v0.1 includes only:

- Two presets (`general`, `it-product`)
- Six de Bono hats
- Configurable 1–6 rounds
- No custom presets, lite mode, or CLI wrapper

Everything else belongs in `Future evolution path` in `README.md`. Do not build until there is a real need.

### Parallelism is mandatory

In orchestrator code/prompts, parallel `Task` calls must be issued in **one message**. Breaking this defeats the purpose of the skill.

### Files on disk are the source of truth

Each round, each hat analysis, concepts, and reports are written to disk **before** the next step. That enables:

- Recovery after session failure
- Artifact inspection
- Reproducibility

### No pinned models in MVP

All `Task` calls omit the `model` parameter. `config/models.yaml` holds structure and `null` values for future use. See `references/models-guide.md`.

## Development workflow

### When changing prompts

1. Edit the template under `prompts/`
2. Read a successful run under `runs/` if available
3. Consider ripple effects on other templates (e.g. if `id` format changes, search all usages)

### When adding a preset

1. Add `config/presets/<name>.yaml` following `general.yaml`
2. If auto-detection is needed, add a `triggers` section with keywords
3. Update `references/presets-guide.md` with when to use it
4. Run a representative task for that preset
5. **Do not edit `SKILL.md`** — it already loads any preset from `config/presets/`

### When changing schemas

1. Update `schemas/*.json`
2. Update `prompts/` templates to match the new schema
3. Update `references/orchestration.md` if validation flow changes
4. **Breaking schema changes break compatibility with existing runs**

### Before committing

1. Ensure no accidental `runs/<...>/` commits (must stay gitignored)
2. Ensure new files are referenced and `SKILL.md` has no orphan links
3. Use the `commit-work` skill for Conventional Commits

## Do not

- **Do not add README / INSTALLATION / CHANGELOG inside the skill bundle.** Anthropic recommends keeping skill context lean; human docs live at repo root (`README.md`).
- **Do not merge roles with hats.** They are orthogonal; see `references/methodology.md`, section “Why NOT Merge Roles With Hats”.
- **Do not add a lite mode in MVP.** For fast brainstorming, use the separate `brainstorming` skill (single agent). This skill targets serious multi-agent runs.
- **Do not hard-code subagent model slugs in MVP.** Cursor allowlists are session-dependent; use Auto.

## Testing the skill

Run locally:

1. Simple task (e.g. “ways to improve my morning routine”) — full workflow completes
2. IT-style task (e.g. “improve onboarding in my app”) — `it-product` preset auto-selected
3. Edge cases: `rounds=1` and `rounds=6`
4. Various `concepts_in_phase_2` sizes (1, 3, 5)

Inspect `runs/` artifacts — especially subagent JSON — for quality and schema compliance.

## Publication

The repository is public on GitHub under the **MIT** license (`LICENSE`).

Further methodology maturity (roadmap):

1. Methodology validated on real tasks
2. Prompts refined after 5+ real runs
3. Edge cases documented

See `README.md`, section “Future evolution path”.

## Skill improvements: human-facing output and report

Feedback from a real run (2026-04): final Markdown was **hard to act on** —
verdicts and Six Thinking Hats tables dominated, not “what should I do and why”.
Below is what to change in the **skill and prompts**, not task-specific content.

### Target shape for `final-report.md` (human)

From the first screen the user should get:

1. **What to do** — prioritized numbered steps (or explicit if/then branches). No methodology jargon as the only content (Six Hats, “pursue-with-caveats” as the sole takeaway).
2. **Why this order** — two to five short sentences: tie to the original ask, main tradeoffs, what was deferred and why.
3. **Visualization where it removes ambiguity** — not decoration: e.g. one Mermaid flow/timeline, a compact “step — why — done-when” table, or a concrete suggestion (GIF, flow screenshot, architecture sketch) when dependencies matter. Hat tables are **not** a substitute for an actionable sequence.

Process detail (hats, tensions, tables) belongs in a **second level** under headings like “Analysis detail” or “How we got here” for readers who only want actions.

The orchestrator (Blue hat) in the final-report prompt should write **“What to do / Why” first**, then optional depth — **when we change the skill**: edits in `prompts/orchestrator-final-report.md`, optionally `schemas/final-report.json`, and a short contract in `SKILL.md`.

### Example opening for a human report (illustration)

Not a universal template — shows desired **shape**: concrete actions in priority order, each with a clear “why”; order comes from synthesis, not Phase 2 concept numbers.

For a task like “organically promote a pet OSS project”, the top might look like:

```markdown
## What to do

1. **Ship a CI demo for promotion** — one reproducible GitHub Actions scenario
   (a “like a user” script) plus a short README demo (terminal recording / GIF).
   First step: credibility before big listings and catalogs.
2. **One anchor README or landing page + optional machine-readable index**
   — after green CI demo, so there is a single entry point and a clear reader path.
3. **Staged outreach to lists** — with pauses and contribution rules, only after
   proof and a hub exist.

## Why this order

Briefly: prove it works and clarify entry, then discovery and community;
otherwise promotion hits an empty README or a broken first run.
```

Optionally add **one** diagram (e.g. Mermaid) with the same steps or dependencies; for product tasks, point to demo/GIF as part of “good visualization” for README readers.

Then an “Analysis detail” block with concept verdicts and hats. **Not changing prompts/schemas yet** — notes only here.

Translating the report to another language **does not fix** bad structure: without an explicit action plan at the top, translation does not help.

### Other pipeline gaps (artifacts)

In a three-round Phase 1 run, raw `runs/<id>/round-3/` files were missing (only `round-1/` and `round-2/`). That breaks “every round persisted before the next step” and recovery. Verify ordering in `SKILL.md`: persist after the final round before synthesis and dedup.

## Related skills

- `~/.claude/skills/brainstorming/` — single-agent dialog brainstorming. The two skills complement each other: brainwriting-635 expands the solution space; brainstorming deepens a chosen direction.

## Global rules

See `~/.claude/CLAUDE.md` and `~/StudioProjects/CLAUDE.md` when present.
