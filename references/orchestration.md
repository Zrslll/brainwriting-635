# Orchestration Reference

Detailed reference for how the orchestrator (Blue hat / main agent) coordinates
subagents, rotates inputs, and recovers from failures.

## Parallelism Discipline

The single most important orchestration rule is: **all subagents in a given
round (Phase 1) or for a given concept (Phase 2) MUST be launched in a single
message containing parallel tool calls.**

Sequential launches negate the entire benefit of multi-agent design:
- 6 sequential subagent calls take ~6x longer than parallel
- Sequential calls inadvertently leak context: the orchestrator may unconsciously
  bias later prompts based on earlier outputs

Each round = one orchestrator message with 6 parallel `Task` calls. Each
concept's Phase 2 = one orchestrator message with 5 parallel `Task` calls.

## Subagent Launch Specification

For each subagent in Phase 1:

```
Task tool call:
  subagent_type: generalPurpose
  description: "<role-name> round <N>"
  prompt: <fully-rendered template from prompts/role-round-1.template.md
           or prompts/role-round-N.template.md with all placeholders replaced>
  model: <NOT SET in MVP - omit the parameter entirely>
```

For each subagent in Phase 2:

```
Task tool call:
  subagent_type: generalPurpose
  description: "<hat-name> on <concept-id>"
  prompt: <fully-rendered template from prompts/hat-prompt.template.md
           with all placeholders replaced>
  model: <NOT SET>
```

## Run Lifecycle

### Initialization

1. Receive task from user
2. Detect or ask for preset (`general` vs `it-product`)
3. Confirm round count (default 3, allow 1-6)
4. Generate `run_id` = `<YYYY-MM-DD>-<slug-of-task>`
5. Create `runs/<run_id>/` directory
6. Write `runs/<run_id>/config.yaml` with chosen settings
7. Show config to user, get confirmation

### Phase 1 Execution

For round N from 1 to total_rounds:

1. **Pre-flight check:**
   - Verify all `runs/<run_id>/round-<N-1>/*.json` files exist (skip for N=1)
   - Read `config/rotation.yaml` for round N mapping
2. **Render prompts:**
   - For each role at position p, render the appropriate template
   - Round 1: use `role-round-1.template.md`
   - Rounds 2..N: use `role-round-N.template.md` with rotation source
3. **Launch:** single message with 6 parallel `Task` calls
4. **Validate:** parse each response as JSON, validate against
   `schemas/round-output.json`
5. **Recover:** if validation fails for a single subagent, relaunch ONLY that
   subagent with a strict reminder about JSON-only output. Max 2 retries.
6. **Save:** write each output to `runs/<run_id>/round-<N>/<role_id>.json`
7. **Status:** brief progress message to user (e.g. "Round 2/3 complete: 18 new ideas")

### Synthesis

After Phase 1 completes:

1. Load all round outputs
2. Run dedup procedure (see `prompts/orchestrator-synthesis.md`)
3. Cluster into concepts
4. Score and rank
5. Write `runs/<run_id>/concepts.json`
6. Present top 5 to user
7. Wait for user selection (which concepts go to Phase 2)

### Phase 2 Execution

For each selected concept:

1. **Render prompts:** 5 hat templates (white, yellow, black, red, green)
2. **Launch:** single message with 5 parallel `Task` calls
3. **Validate:** each response against `schemas/hat-analysis.json`
4. **Save:** `runs/<run_id>/hat-analysis/<concept_id>/<hat>.json`
5. **Synthesize:** Blue hat (orchestrator) writes per-concept verdict

### Final Report

1. Aggregate all hat analyses
2. Render final report (see `prompts/orchestrator-final-report.md`)
3. Write `runs/<run_id>/final-report.json` (structured)
4. Write `runs/<run_id>/final-report.md` (human-readable)
5. Show summary to user with file path

## Failure Recovery

### Subagent returns malformed JSON

- Save the raw response to `runs/<run_id>/round-<N>/<role_id>.raw.txt`
- Relaunch the same subagent with an additional reminder:
  "Your previous response failed JSON validation. Output ONLY a single
  JSON object matching the schema. No prose, no markdown fences."
- Maximum 2 retries per subagent. After that, mark the role as failed for
  this round and proceed (the round will have 5 outputs instead of 6).

### Subagent returns ideas with wrong `builds_on` references (rounds 2..N)

- A `builds_on` ID must reference an actual idea from the previous round
- If invalid, relaunch with: "Your idea references `<bad_id>` which does not
  exist in the input. Use one of these IDs: [list]."

### Web search fails

- Subagents should proceed without web search if it fails
- The `sources` field will be empty - that's acceptable
- Do not block the round on web search failures

### User abandons before Phase 2

- Phase 1 artifacts remain in `runs/<run_id>/`
- A subsequent session can resume by reading `concepts.json` and asking the
  user which concepts to validate

### Orchestrator runs out of context

- This is a hard failure - the run cannot continue
- Files on disk preserve all state
- A fresh orchestrator can resume from the last completed round by reading
  `runs/<run_id>/` and identifying the next missing round

## Performance and Cost Notes

### Subagent count per run

| Configuration | Phase 1 calls | Phase 2 calls | Total |
|---|---|---|---|
| 3 rounds, 3 concepts | 18 | 15 | 33 |
| 3 rounds, 5 concepts | 18 | 25 | 43 |
| 6 rounds, 3 concepts | 36 | 15 | 51 |
| 6 rounds, 5 concepts | 36 | 25 | 61 |

Each subagent call may also trigger 1-2 web searches. Plan for ~50-120 web
search operations per full run.

### Wall clock

- Each round: limited by the slowest subagent (parallel execution). Typically 30-90 seconds per round.
- Phase 1 total: ~2-5 minutes for 3 rounds, ~5-10 minutes for 6 rounds
- Phase 2: ~1-2 minutes per concept (5 hats in parallel)
- Synthesis and report: ~1 minute orchestrator work

Total realistic time for default 3-round / 3-concept run: **~10-15 minutes**.

### Cost discipline

The skill should warn the user upfront about expected scale ("This will run
~33 subagent calls and may take 10-15 minutes"). Get explicit confirmation
before launching.

## Run Directory Layout

```
runs/<run_id>/
├── config.yaml                        # what was configured for this run
├── round-1/
│   ├── pragmatist.json
│   ├── contrarian.json
│   ├── ux_advocate.json
│   ├── engineer.json
│   ├── business.json
│   └── visionary.json
├── round-2/
│   └── ... (same structure)
├── round-3/
│   └── ... (same structure)
├── concepts.json                      # output of synthesis stage
├── hat-analysis/
│   ├── concept-1/
│   │   ├── white.json
│   │   ├── yellow.json
│   │   ├── black.json
│   │   ├── red.json
│   │   └── green.json
│   └── concept-2/
│       └── ...
├── final-report.json                  # structured final output
└── final-report.md                    # human-readable final output
```

This directory is gitignored. It is intended as local working state, not
shared artifact (though users may choose to share specific reports).
