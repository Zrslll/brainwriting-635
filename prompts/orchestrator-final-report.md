# Orchestrator: Final Report Procedure (End of Phase 2)

After all hat analyses complete for all selected concepts, the orchestrator
(wearing the Blue hat) produces the final report.

## Inputs

- `runs/<run_id>/concepts.json` (the chosen concepts)
- `runs/<run_id>/hat-analysis/<concept_id>/<hat>.json` for each concept and each
  of the 5 colored hats (white, yellow, black, red, green)
- The original task statement
- Run config (preset, rounds, models used)

## Procedure

### Step 1: Per-concept synthesis (Blue hat work)

For each concept, do the following:

1. **Read all 5 hat analyses** for that concept
2. **Identify tensions** between hats. Common patterns:
   - Yellow says "high value" but Black says "regulatory blocker"
   - White says "no data exists" but Red says "feels right"
   - Green offers an alternative that resolves Black's main risk
3. **Distill each hat's verdict** into one paragraph (use the `summary` field as a starting point but rewrite for the report)
4. **Decide a verdict** from the enum:
   - `strong-pursue` - All hats positive or risks easily addressed; clear next steps
   - `pursue-with-caveats` - Net positive but specific risks must be managed
   - `explore-further` - Insufficient information; needs more research before commitment
   - `park` - Interesting but not now; revisit when context changes
   - `reject` - Risks outweigh benefits or fundamental flaw discovered
5. **Write verdict reasoning** that cites the most decisive hat findings (not generic prose)
6. **List concrete next steps** if pursuing - specific actions, not "explore more"

### Step 2: Cross-concept summary

After all concepts have verdicts, write the top-level summary:

- **Recommendation** - which concept(s) to pursue, or "no clear winner" if applicable
- **Reasoning** - why this recommendation, comparing the concepts
- **Statistics** - total ideas generated, after dedup, concepts analyzed

### Step 3: Generate the report files

#### Structured: `runs/<run_id>/final-report.json`

Match `schemas/final-report.json` exactly. This is for programmatic consumers.

#### Human: `runs/<run_id>/final-report.md`

Markdown report optimized for human reading. Suggested structure:

```markdown
# Brainwriting Report: {{TASK_TITLE}}

**Date:** {{DATE}}
**Preset:** {{PRESET}}
**Rounds:** {{ROUNDS}}
**Ideas generated:** {{N_TOTAL}} -> {{N_DEDUP}} after dedup -> {{N_CONCEPTS}} analyzed

## Recommendation

{{TOP_LEVEL_RECOMMENDATION}}

## Concept 1: {{CONCEPT_TITLE}}

**Verdict:** {{VERDICT}}

{{SYNTHESIS_PARAGRAPH}}

### Six Hats Analysis

**White (facts):** {{WHITE_SUMMARY}}

**Yellow (benefits):** {{YELLOW_SUMMARY}}

**Black (risks):** {{BLACK_SUMMARY}}

**Red (intuition):** {{RED_SUMMARY}}

**Green (alternatives):** {{GREEN_SUMMARY}}

### Tensions

- {{TENSION_1}}
- {{TENSION_2}}

### Next Steps

1. {{NEXT_STEP_1}}
2. {{NEXT_STEP_2}}

---

## Concept 2: ...

(repeat for each concept)

---

## Run Metadata

- Started: {{START_TIME}}
- Completed: {{END_TIME}}
- Models used: {{MODELS_TABLE}}
- Run directory: `runs/{{RUN_ID}}/`
```

### Step 4: Present to user

Show:
1. Top-level recommendation in chat
2. Verdict for each concept (one line each)
3. Path to the full report file

Offer follow-ups:
- Translate the report to a specific language
- Deep-dive on a specific concept
- Export to a specific format (Notion, Confluence, etc.)
- Run a second brainwriting on a refined sub-question

## Anti-Patterns

- **Generic verdicts.** "This concept has potential" tells the user nothing. Be specific about what is strong and what is weak.
- **Hiding tensions.** If Yellow and Black disagree sharply, surface that disagreement in the report. Tensions are signal.
- **Vague next steps.** "Validate with users" is forbidden. "Run a 5-user concierge test of the voice flow within 1 week" is good.
- **Equally praising every concept.** If you cannot pick a recommendation, the run failed. Say so honestly and suggest what would help.
