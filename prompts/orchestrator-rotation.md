# Orchestrator: Letter Rotation Procedure

This document describes how the orchestrator (Blue hat / main agent) prepares
inputs for rounds 2..N of Phase 1.

## Why This Matters

In the 6-3-5 method, ideas evolve as they pass between participants. Each
subagent in round N receives ideas from a SPECIFIC other subagent's previous
round, not from a random pool. The mapping is deterministic.

**Do not compute the rotation in your head or via prose reasoning.** Read the
mapping directly from `config/rotation.yaml`.

## Procedure for Round N (where N >= 2)

### Step 1: Determine the role order

The role order is **fixed by the chosen preset**. Read it from the preset YAML's
`roles[]` list. Example for `general`:

```
position 0: pragmatist
position 1: contrarian
position 2: ux_advocate
position 3: engineer
position 4: business
position 5: visionary
```

### Step 2: Read the rotation mapping for round N

From `config/rotation.yaml`, look up `rounds.<N>.mapping`. The mapping is:
`<destination_position>: <source_position>`.

Example for round 2:
```yaml
rounds:
  2:
    mapping:
      0: 5
      1: 0
      2: 1
      3: 2
      4: 3
      5: 4
```

This means: the role at position 0 receives from position 5, position 1 from
position 0, and so on (shift by 1).

### Step 3: Build per-role inputs

For each destination role at position `p`:

1. Look up `source_position = mapping[p]`
2. Identify the role at `source_position` in the preset's role list
3. Load that source role's output from the **immediately previous round's** file:
   `runs/<run_id>/round-<N-1>/<source_role_id>.json`
4. Pass that complete `RoundOutput` JSON as `INCOMING_IDEAS_JSON` in the
   round-N prompt template for the destination role

### Step 4: Launch all 6 subagents in parallel

Send a single message containing all 6 Task tool calls. Do NOT launch them
sequentially - that defeats the purpose of parallelism.

### Step 5: Save outputs

After all 6 subagents return, save each output to:
`runs/<run_id>/round-<N>/<role_id>.json`

Validate each output against `schemas/round-output.json` before saving. If a
subagent returned malformed JSON, retry that single subagent (not all six).

## Verification

Before launching round N, verify:

- [ ] You have all 6 outputs from round N-1 saved on disk
- [ ] You read `rounds.<N>.mapping` from `config/rotation.yaml`, not from memory
- [ ] You loaded `INCOMING_IDEAS_JSON` from the **previous** round (N-1), not from round 1
- [ ] All 6 Task calls are in a single message for parallel execution
- [ ] Each prompt includes the correct `SOURCE_ROLE_NAME` for the destination role

## Common Mistakes to Avoid

1. **Always pulling from round 1.** The point of rotation is that ideas evolve. Round 3 should build on round 2's evolved ideas, not on the originals.
2. **Computing rotation in prose.** "Position 3 in round 2 receives from position 2 by shift-by-1 logic..." - just read the YAML.
3. **Sending one Task call at a time.** Sequential subagent calls waste the parallelism benefit and slow the run by 6x.
4. **Skipping schema validation.** A malformed round breaks all subsequent rounds.
