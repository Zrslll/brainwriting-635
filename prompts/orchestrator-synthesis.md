# Orchestrator: Synthesis Procedure (End of Phase 1)

After all rounds of Phase 1 complete, the orchestrator must transform the raw
idea pool (typically 18-108 ideas) into a small set of distinct concepts that
can be analyzed in Phase 2.

## Inputs

- All `runs/<run_id>/round-<N>/<role_id>.json` files from rounds 1..N
- The original task statement
- The chosen preset

## Procedure

### Step 1: Load and flatten

Read every `RoundOutput` JSON from every round. Flatten into a single list of
`Idea` objects. Track origin (role + round) for each.

### Step 2: Deduplicate

Cluster ideas by **semantic similarity**, not by literal text. Two ideas are the
same concept when:
- They propose essentially the same intervention or change
- They would have the same implementation regardless of phrasing
- They would succeed or fail under the same conditions

For each duplicate cluster:
- Keep the most specific and well-developed phrasing as the canonical version
- Record the contributing idea IDs in a `merged_from` field
- Synthesize a unified `hypothesis` if the originals had complementary detail
- Combine the `risk` fields, keeping the most severe and specific

### Step 3: Cluster into concepts

After dedup, group remaining ideas by THEME (not by role). A "concept" is a
coherent direction that may bundle 1-5 deduplicated ideas. Examples:
- "Voice-first onboarding" might bundle ideas from UX, Product, and Engineering
- "Subscription with usage tiers" might bundle Business + Pragmatist ideas

Aim for **5-10 concepts** total. If you have far more, you have not deduplicated
enough. If far fewer, the brainwriting was too narrow.

### Step 4: Score each concept

For each concept, score on 4 dimensions (1-5 scale, integers only):

- **Impact** - if this works, how much does it move the needle?
- **Effort** - how much work to build a meaningful version? (1 = huge effort, 5 = quick)
- **Novelty** - how different is this from default approaches?
- **Fit** - how well does this match the task statement and constraints?

Compute a composite priority. Suggested formula:
`priority = (impact * 2) + effort + novelty + (fit * 2)`

### Step 5: Select top concepts

Sort by composite priority. Identify the top 5 (or fewer if there are fewer).
These go to Phase 2 unless the user picks differently at the checkpoint.

### Step 6: Write `concepts.json`

Save to `runs/<run_id>/concepts.json` as an array of concept objects:

```json
[
  {
    "id": "concept-1",
    "title": "Concept title",
    "synthesis": "Unified description bundling the source ideas.",
    "hypothesis": "What this concept claims will happen.",
    "merged_from": ["pragmatist-r1-1", "engineer-r2-2", ...],
    "score": {"impact": 4, "effort": 3, "novelty": 5, "fit": 4},
    "priority": 21,
    "rank": 1
  }
]
```

### Step 7: Present to user

Show the top 5 concepts in human-readable form. Ask:

> "These are the strongest concepts from {{N_TOTAL}} generated ideas
> ({{N_DEDUP}} after dedup). Which ones should I run through the Six Hats
> validation in Phase 2?
>
> Options:
> - Type concept IDs (e.g. '1, 3, 4')
> - 'auto' = top 3
> - 'all' = all five
> - 'add: <description>' = add your own concept to the analysis"

Wait for user response before launching Phase 2.

## Anti-Patterns

- **Skipping dedup.** Running Phase 2 on 50 raw ideas is computationally wasteful and dilutes signal.
- **Scoring everything 4 or 5.** If all concepts look great, you are not being honest. Force a real distribution.
- **Letting role bias dictate clustering.** A "great" idea from the Visionary that no other role validated is suspicious, not visionary.
- **Auto-skipping the checkpoint.** Always ask the user. The whole point is human judgment on which directions matter.
