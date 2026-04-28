# brainwriting-635

A Cursor / Claude Skill for **multi-agent structured ideation** that combines
two well-known thinking methods into a single two-phase workflow:

- **Phase 1 — 6-3-5 Brainwriting** (divergence): 6 role-based subagents
  generate ideas in parallel across N rounds with letter rotation between rounds
- **Phase 2 — Six Thinking Hats** (convergence): 5 hat-based subagents
  validate each top concept in parallel; the orchestrator wears the Blue hat
  to synthesize

## Status

**v0.1**

## Why This Exists

Existing brainstorming skills are either single-agent (one Claude in a dialog)
or shallow imitations of multi-agent methods (one Claude pretending to be six
people in one context). This skill is different in three ways:

1. **Real parallelism via the Task tool.** Six subagents launched in a single
   message, each with isolated context.
2. **Hybrid methodology.** 6-3-5 for breadth, Six Hats for depth, with a
   user checkpoint between phases.
3. **Strict contracts.** JSON schemas for every subagent output enable
   programmatic dedup, scoring, and report generation rather than free-form prose.

To our knowledge, no existing public skill implements 6-3-5 with real
parallel subagents AND combines it with Six Hats validation.

## Architecture

```
SETUP -> PHASE 1 (rounds) -> SYNTHESIS -> CHECKPOINT -> PHASE 2 (hats) -> FINAL REPORT
```

### Phase 1: 6-3-5 Brainwriting

For each round:
- 6 role subagents launch in parallel (one message, six `Task` calls)
- Each role generates 1-3 ideas through their distinct lens
- Round 1: ideas from scratch + web search
- Rounds 2..N: ideas develop from another role's previous output (letter rotation)

### Synthesis (Orchestrator)

- Flatten ~18-108 raw ideas
- Semantic dedup
- Cluster by theme into 5-10 concepts
- Score on impact / effort / novelty / fit
- Present top 5 to user

### Checkpoint

User selects which concepts to validate in Phase 2 (1 to 5 concepts).

### Phase 2: Six Thinking Hats

For each selected concept:
- 5 hat subagents launch in parallel (White, Yellow, Black, Red, Green)
- White hat uses web search to ground facts
- Each hat operates strictly in its own mode

### Final Report

- Per-concept synthesis with verdict (`strong-pursue`, `pursue-with-caveats`, `explore-further`, `park`, `reject`)
- Cross-concept top-level recommendation
- Both structured JSON and human-readable Markdown outputs

## Repository Structure

```
brainwriting-635/
├── SKILL.md                  Main skill file (loaded by Cursor/Claude)
├── CLAUDE.md                 Maintainer guidelines (English); not part of the skill runtime
├── README.md                 This file
├── LICENSE                   MIT License
├── .gitignore
│
├── config/
│   ├── presets/
│   │   ├── general.yaml      Pragmatist, Contrarian, UX Advocate, Engineer, Business, Visionary
│   │   └── it-product.yaml   PM, UX/UI, Frontend, Backend, QA, Growth
│   ├── hats.yaml             Six Thinking Hats definitions
│   ├── rotation.yaml         Letter rotation mapping per round
│   └── models.yaml           Model pinning placeholder (all null in MVP)
│
├── prompts/
│   ├── role-round-1.template.md      Phase 1 initial generation
│   ├── role-round-N.template.md      Phase 1 development rounds
│   ├── hat-prompt.template.md        Phase 2 hat analysis
│   ├── orchestrator-rotation.md      How to rotate inputs in rounds 2..N
│   ├── orchestrator-synthesis.md     How to dedup, cluster, score, rank
│   └── orchestrator-final-report.md  How to produce the final report
│
├── schemas/
│   ├── idea.json
│   ├── round-output.json
│   ├── hat-analysis.json
│   └── final-report.json
│
├── references/
│   ├── methodology.md         Why the methods work and why we combine them
│   ├── orchestration.md       Parallelism, failure recovery, run lifecycle
│   ├── models-guide.md        Current MVP policy and future model pinning
│   └── presets-guide.md       Preset selection and role design rationale
│
└── runs/                      Gitignored. Per-run artifacts.
    └── <YYYY-MM-DD>-<topic>/
        ├── config.yaml
        ├── round-1/<role>.json
        ├── ...
        ├── concepts.json
        ├── hat-analysis/<concept>/<hat>.json
        ├── final-report.json
        └── final-report.md
```

**CLAUDE.md** is for maintainers (conventions, how to change prompts/schemas, AI-assisted development rules). It is **not** loaded when you use the skill—only `SKILL.md`, `config/`, `prompts/`, `schemas/`, and `references/` are needed for orchestration. You can symlink the repo as documented below without relying on `CLAUDE.md`.

Optionally keep a local **`CLAUDE.ru.md`** as a Russian mirror of the same content; that filename is **gitignored** so only the English `CLAUDE.md` ships with the repository.

## Installation

Clone this repository, then symlink the skill into your local Claude / Cursor skills directory:

```bash
git clone https://github.com/Zrslll/brainwriting-635.git
cd brainwriting-635
ln -s "$(pwd)" ~/.claude/skills/brainwriting-635
```

After installation, the skill triggers on phrases like:
- "run brainwriting on X"
- "do 6-3-5 on X"
- "запусти брейнрайтинг по X"
- "мозговой штурм с агентами по X"

The skill does NOT auto-trigger on generic phrases like "brainstorm" or "ideas" -
that would consume too many resources unintentionally.

## Scale

| Configuration | Subagent calls |
|---|---|
| 3 rounds, 3 concepts (default) | 33 |
| 3 rounds, 5 concepts | 43 |
| 6 rounds, 3 concepts | 51 |
| 6 rounds, 5 concepts | 61 |

Each subagent may also trigger 1-2 web searches. The orchestrator warns the
user about expected scale before launching.

## Models

**v0.1 MVP does not pin specific model slugs.** All `Task` calls launch
without the `model` parameter, allowing Cursor to use its session default
(typically Auto).

The `config/models.yaml` file exists with the structure ready for future
versions. See `references/models-guide.md` for the rationale and migration
path when model pinning becomes valuable.


## Methodology Credits

- **6-3-5 Brainwriting** — Bernd Rohrbach (1968), "Kreativ nach Regeln —
  Methode 635, eine neue Technik zum Lösen von Problemen"
- **Six Thinking Hats** — Edward de Bono (1985), "Six Thinking Hats"

This skill adapts both methods for LLM multi-agent orchestration. The
adaptations and the rationale for combining them are documented in
`references/methodology.md`.

## License

[MIT](LICENSE). See the license file for full terms.
