# Methodology

This skill combines two well-known structured-thinking methods into a single
two-phase workflow optimized for multi-agent orchestration.

## Phase 1: 6-3-5 Brainwriting (Divergence)

### Origin

Brainwriting 6-3-5 was created by Bernd Rohrbach in 1968 as a written
alternative to verbal brainstorming. The numbers stand for: 6 participants,
3 ideas each, 5-minute rounds, with sheets passed around the circle.

### Why It Works

- **Parallelism eliminates groupthink.** No participant hears others' ideas in
  real time during a round; every contribution is independent.
- **Builds on prior thought.** Subsequent rounds force participants to develop
  what others wrote, producing emergent ideas no individual would have reached.
- **Time-boxed.** Forces decisions over endless deliberation.

### Adaptation for LLM Subagents

The classic method translates well to LLMs but requires four adaptations:

1. **No time pressure.** LLMs do not need 5-minute limits. Quality over the
   "1-3 ideas" output range is enforced via prompt instead.
2. **Asynchronous rotation.** "Passing the sheet" becomes "orchestrator passes
   one subagent's output as input to another in the next round."
3. **Configurable rounds.** Default is 3 rounds (sweet spot of cost/quality);
   max is 6 (the canonical 6-3-5).
4. **Role-based diversity.** Identical LLM agents would think identically.
   Distinct ROLES (different system prompts, lenses, typical questions) create
   genuine cognitive diversity even on a single underlying model.

### Output of Phase 1

Up to `6 * 3 * N rounds` raw ideas (e.g. 54 for 3 rounds, 108 for 6 rounds).
After dedup and clustering: typically 5-10 distinct concepts.

## Phase 2: Six Thinking Hats (Convergence)

### Origin

Six Thinking Hats was developed by Edward de Bono (1985) as a parallel-thinking
method to focus a group on one mode of reasoning at a time, eliminating
back-and-forth confusion.

### The Hats

- **White** - facts, data, what is known
- **Yellow** - benefits, value, optimism
- **Black** - risks, critique, what could go wrong
- **Red** - emotion, intuition, gut feeling
- **Green** - creativity, alternatives, what else could work
- **Blue** - process management, synthesis, meta

### Why It Works

- **Forces full-spectrum analysis.** Naturally critical thinkers must wear
  Yellow; naturally optimistic thinkers must wear Black. No mode is skipped.
- **Surfaces tensions explicitly.** When Yellow and Black disagree on the same
  concept, that disagreement IS the signal worth investigating.
- **Separates evaluation from generation.** Reduces the "kill the idea before
  it grows" problem of unstructured discussion.

### Adaptation for LLM Subagents

- **5 colored hats run in parallel as subagents** for each concept.
- **Blue hat is the orchestrator** (the main agent), responsible for synthesis
  and process control.
- Each hat has strict mode constraints in its prompt - White cannot offer
  predictions, Black cannot propose solutions, etc. This prevents the common
  LLM failure mode of "balanced both-sides analysis" that destroys the value
  of single-mode thinking.

## Why Combine the Two Methods

The two methods solve different problems:

| | 6-3-5 Brainwriting | Six Hats |
|---|---|---|
| **Purpose** | Generate many distinct ideas | Deeply evaluate one concept |
| **Mode** | Divergent | Convergent |
| **Output** | Quantity (54+ ideas) | Quality (full-spectrum analysis) |
| **Best for** | "We need options" | "Should we do this?" |

Used together, they cover the full creative pipeline:
1. Generate broadly (6-3-5)
2. Cluster into concepts
3. Evaluate deeply (Six Hats)
4. Decide

## Why NOT Merge Roles With Hats

A tempting design is to map roles 1:1 to hats (Contrarian = Black, Visionary =
Green, etc.). This is rejected because:

- **Roles and hats are orthogonal axes.** Roles are domain expertise
  (Engineer thinks about systems); hats are cognitive modes (Black thinks
  about risk). An Engineer-in-Black-hat thinks about technical risks. A
  Business-in-Black-hat thinks about market risks. Both are valuable.
- **Each hat applied with one role lens loses 5 other lenses.** A "Contrarian
  Black hat" never benefits from a Yellow-hat Engineer perspective.
- **The two methods serve different phases.** Roles maximize divergence in
  generation. Hats maximize completeness in evaluation. Conflating them
  collapses the funnel.

## Tradeoffs and Limitations

- **Cost scales with rounds.** A 6-round full-canon run with 5-hat analysis
  on 5 concepts uses 36 + 25 = 61 subagent calls. Default 3-round / 3-concept
  runs use 18 + 15 = 33 calls. Plan accordingly.
- **LLM "diversity" is partially synthetic.** Different roles on the same
  model still share underlying training data biases. This is mitigated by
  role-specific prompts but not eliminated.
- **Web search adds variance.** Subagents that find different sources may
  produce different ideas than subagents that search differently. This is
  a feature (real research influence) but it makes runs non-reproducible.
- **Dedup quality limits final report quality.** If the orchestrator's
  semantic dedup is sloppy, Phase 2 wastes effort on near-duplicate concepts.
