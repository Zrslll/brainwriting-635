---
name: brainwriting-635
description: "Multi-agent structured ideation combining the 6-3-5 brainwriting method (Phase 1, divergent generation through 6 role-based subagents) with Edward de Bono's Six Thinking Hats (Phase 2, convergent validation through 5 hat-based subagents per concept). Use ONLY when explicitly invoked - this skill launches 30-60 parallel subagents and takes 10-15 minutes per run. Triggers: 'run brainwriting', 'do 6-3-5 brainwriting', 'multi-agent brainstorm', 'провести брейнрайтинг', 'запусти 6-3-5', 'мозговой штурм с агентами'."
---

# Brainwriting 6-3-5 + Six Hats

Two-phase structured ideation skill that orchestrates parallel subagents to
generate, develop, and validate ideas at scale.

## When To Use

**Use this skill ONLY when explicitly invoked** by phrases like:
- "Run brainwriting on X"
- "Do 6-3-5 on X"
- "Run a multi-agent brainstorm on X"
- "Запусти брейнрайтинг по X"

**Do NOT use this skill** for:
- Quick ideation (use the regular `brainstorming` skill instead)
- Single-question Q&A
- Code-writing tasks
- When the user has not explicitly asked for the 6-3-5 / brainwriting workflow

This skill is expensive (30-60 subagent calls per run, 10-15 min wall clock).
Confirm with the user before launching.

## Workflow Overview

```
SETUP -> PHASE 1 (rounds) -> SYNTHESIS -> CHECKPOINT -> PHASE 2 (hats) -> FINAL REPORT
```

You (the main agent) act as the **Blue hat / orchestrator** throughout.

## Setup

### Step 1: Confirm the task

If the user did not provide a clear task statement, ask for one. The task
should be a brainstormable problem or question, e.g. "How do we improve
checkout conversion?" rather than "Tell me about checkout."

### Step 2: Select preset

Read the user's task statement. Scan for IT keywords (see `triggers` section
in `config/presets/it-product.yaml`).

- **2+ IT keywords match:** propose `it-product` preset
- **0-1 IT keywords match:** propose `general` preset

State your choice and reasoning to the user. Allow override.

For deeper guidance on preset selection, read `references/presets-guide.md`.

### Step 3: Confirm rounds

Default: **3 rounds**. Ask the user if they want a different count (1-6).

- 1 round = quick fan-out, no idea evolution (50 ideas total of typical 18 generated)
- 3 rounds = balanced default (~54 ideas before dedup)
- 6 rounds = full canonical 6-3-5 (~108 ideas before dedup, 2x cost and time)

### Step 4: Show config and get confirmation

Show the user:
- Selected preset and the 6 roles
- Round count
- Estimated subagent calls (`6 * rounds + 5 * concepts_in_phase_2`)
- Estimated wall clock (~10-15 min for default 3-round / 3-concept run)

Get explicit "yes" before proceeding.

### Step 5: Initialize run directory

Create `runs/<YYYY-MM-DD>-<topic-slug>/`. Write `config.yaml` with the chosen
settings, task statement, and start timestamp.

## Phase 1: Brainwriting Rounds

For each round N from 1 to total_rounds:

1. **Pre-flight:** verify previous round outputs exist (skip for N=1)
2. **Render prompts:**
   - Round 1: use `prompts/role-round-1.template.md`
   - Rounds 2..N: use `prompts/role-round-N.template.md` with rotation source
3. **Read rotation:** for rounds 2..N, read `config/rotation.yaml` to determine
   which role's previous output goes to which destination role
4. **Launch 6 subagents in parallel:** ONE message containing 6 `Task` tool calls
5. **Validate:** parse each response as JSON against `schemas/round-output.json`
6. **Save:** write each output to `runs/<run_id>/round-<N>/<role_id>.json`
7. **Status update:** brief progress message to user

**Critical:** all 6 Task calls in a single message. Sequential launches lose
the entire benefit of multi-agent design.

For full rotation procedure including failure recovery, read
`prompts/orchestrator-rotation.md`.

For launch specifications and parallelism rules, read `references/orchestration.md`.

## Synthesis (After Phase 1)

After all rounds complete:

1. Load all `round-N/<role>.json` files
2. Flatten to a single idea pool
3. **Deduplicate** by semantic similarity (not literal text)
4. **Cluster** into 5-10 distinct concepts (group by theme, not by role)
5. **Score** each concept on impact / effort / novelty / fit (1-5 scale)
6. Rank by composite priority
7. Write `runs/<run_id>/concepts.json`
8. Present **top 5 concepts** to user

Detailed procedure in `prompts/orchestrator-synthesis.md`.

## Checkpoint With User

After presenting top 5 concepts, ask:

> "Which concepts should I run through the Six Hats validation?
> Options:
> - Type concept IDs (e.g. '1, 3, 4')
> - 'auto' = top 3
> - 'all' = all five
> - 'add: <description>' = add your own concept first"

**Wait for user response. Do NOT auto-proceed.** The whole point of this
checkpoint is human judgment on direction.

## Phase 2: Six Hats Validation

For each user-selected concept:

1. **Render 5 hat prompts** from `prompts/hat-prompt.template.md`:
   - White (facts, web search REQUIRED)
   - Yellow (benefits)
   - Black (risks)
   - Red (intuition)
   - Green (alternatives)
2. **Launch 5 subagents in parallel:** ONE message containing 5 `Task` tool calls
3. **Validate** each response against `schemas/hat-analysis.json`
4. **Save:** `runs/<run_id>/hat-analysis/<concept_id>/<hat>.json`

The Blue hat (you, the orchestrator) does NOT run as a subagent. You synthesize
the 5 colored hats yourself.

## Final Report

After Phase 2 completes for all selected concepts:

1. **Per-concept synthesis:** for each concept, identify tensions between hats,
   distill each hat's verdict, decide a verdict from the enum
   (`strong-pursue` / `pursue-with-caveats` / `explore-further` / `park` / `reject`),
   list concrete next steps
2. **Cross-concept summary:** top-level recommendation comparing concepts
3. **Write structured report:** `runs/<run_id>/final-report.json` matching `schemas/final-report.json`
4. **Write human report:** `runs/<run_id>/final-report.md`
5. **Present to user:** show top-level recommendation and per-concept verdicts in chat, with the path to the full report file

Detailed procedure in `prompts/orchestrator-final-report.md`.

## Critical Constraints

### No imitation

You MUST use the `Task` tool to launch real subagents. Do NOT simulate "as if"
6 different roles in a single context window. Imitation defeats the entire
purpose of this skill - parallel subagents with isolated contexts are what
generates genuine cognitive diversity.

### Parallelism

All subagents in a given round (Phase 1) or for a given concept (Phase 2) MUST
launch in a single message containing parallel `Task` calls. Sequential calls
add unnecessary latency and risk context contamination.

### No model pinning in MVP

Launch all `Task` calls **without the `model` parameter**. The `config/models.yaml`
file exists as scaffolding for future versions but currently contains only `null`
values. Cursor will use the session's default model.

### Schema enforcement

Every subagent output must validate against its JSON schema. Malformed responses
trigger a single retry with stricter instructions. After 2 failed retries for
the same subagent, mark that subagent as failed and proceed (the round will
have one fewer output).

### File-first state

Save every round and every hat analysis to disk BEFORE moving to the next step.
This makes the run resumable if anything fails.

### English everywhere

All prompts, schemas, configs, and reports are in English. The orchestrator
may translate the final report to another language if the user requests it
explicitly after the run completes.

### Always checkpoint

Never auto-skip the checkpoint between Phase 1 and Phase 2. The user's
selection of which concepts to validate is a critical signal that should not
be replaced by orchestrator judgment.

## File Reference

When you need detail on a specific aspect, read these files (in order of
likely need):

- `prompts/orchestrator-rotation.md` - how to rotate inputs in rounds 2..N
- `prompts/orchestrator-synthesis.md` - how to dedup, cluster, score, rank
- `prompts/orchestrator-final-report.md` - how to produce the final report
- `references/orchestration.md` - parallelism, failure recovery, run lifecycle
- `references/methodology.md` - why the methods work and why we combine them
- `references/presets-guide.md` - preset selection and role design rationale
- `references/models-guide.md` - current MVP policy and future model pinning

Configs to read at runtime:

- `config/presets/general.yaml` or `config/presets/it-product.yaml` - role definitions for the chosen preset
- `config/hats.yaml` - hat definitions for Phase 2
- `config/rotation.yaml` - letter rotation mapping per round

Schemas for output validation:

- `schemas/round-output.json` - what subagents return per round
- `schemas/hat-analysis.json` - what hat subagents return per concept
- `schemas/final-report.json` - what the final report contains

Templates to render with placeholders before passing to `Task`:

- `prompts/role-round-1.template.md` - Phase 1 round 1 generation
- `prompts/role-round-N.template.md` - Phase 1 development rounds
- `prompts/hat-prompt.template.md` - Phase 2 hat analysis

## License

Released under the [MIT License](LICENSE) at the repository root.
