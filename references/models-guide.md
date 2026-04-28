# Models Guide

This document explains the model strategy for the skill, both the current MVP
behavior and the rationale for future evolution.

## MVP Behavior (v0.1)

**The skill does NOT pin specific model slugs to subagents.** All `Task` tool
calls are launched without the `model` parameter, allowing Cursor to use its
session default (typically Auto).

### Why Not Pin Models in MVP

1. **Cursor's Task tool whitelist is session-dependent.** The set of models
   available for explicit pinning varies by user's plan, workspace settings,
   and current Cursor version. A skill that hardcodes "use gpt-5 for Black hat"
   may silently fail or refuse to launch on a session where gpt-5 is not in
   the whitelist.

2. **Cognitive diversity comes mainly from prompts, not models.** Six different
   role prompts on the same underlying model produce meaningfully different
   outputs. Six identical role prompts on six different models would produce
   slightly different phrasings of the same thinking. The role variety is the
   bigger lever.

3. **Model slugs change.** A skill that depends on `claude-opus-4.7` will break
   when that slug is renamed or deprecated. Auto/default sidesteps this.

4. **True model diversity is better solved at the CLI layer.** The Phase 3
   evolution path is a standalone CLI that calls Anthropic, OpenAI, and Gemini
   APIs directly. That layer can guarantee a specific model mix because it
   does not depend on Cursor's Task tool.

### What This Means for Output Variety

On a single underlying model, output diversity comes from:

- **Distinct system prompts per role** (lens, typical questions, things to avoid)
- **Different web search results** that ground different ideas
- **Random sampling** in the model's generation process
- **Cross-round rotation** that forces each subagent to develop ideas from a
  different colleague each round

This is sufficient for the MVP's goal: validate that the methodology produces
useful brainwriting outputs.

## Future: When Model Pinning Becomes Worth It

Model pinning would add value in scenarios such as:

| Subagent | Why a Specific Model Helps |
|---|---|
| Black hat | Models that are more willing to be direct/critical (varies by tuning) |
| Red hat | Models with stronger emotional/empathy reasoning |
| Green hat | Models that produce more lateral, unconventional outputs |
| White hat | Models with stronger factual recall and citation discipline |
| Visionary role | Models that are less risk-averse about ambitious claims |

When the skill matures and there is a stable set of slugs available in Cursor,
`config/models.yaml` can be populated with those slugs.

## Forward-Compatible Design

The architecture is ready for model pinning without rework:

1. **`config/models.yaml`** already has the structure (currently all `null`)
2. **The orchestrator's launch procedure** already includes a step to "check
   if a model is configured for this role/hat and pass it to Task tool;
   if null, omit the model parameter"
3. **The final report's `models_used` field** already exists in the schema
   to document what was actually used

To enable pinning later, the change is:
1. Populate slugs in `config/models.yaml`
2. Add availability check in the orchestrator (see Phase 2 evolution in README)
3. No prompt changes, no schema changes, no workflow changes

## How to Configure (Future Use)

When you do want to pin models, edit `config/models.yaml`:

```yaml
phase_1_roles:
  pragmatist: claude-sonnet-4.7    # example only - use slugs available in your Cursor session
  contrarian: gpt-5
  visionary: gemini-2.5-pro
  # leave others null to use Auto

phase_2_hats:
  black: gpt-5
  red: claude-opus-4.7
  green: gemini-2.5-pro
```

The orchestrator should:
1. Verify each non-null slug is in the current Task tool whitelist
2. For unavailable slugs: warn the user, fall back to Auto, log the substitution
   in `final-report.json` `config.models_used`
3. Never silently substitute a different specific model

## Phase 3 Standalone CLI Note

The standalone CLI mode (see project README's evolution path) bypasses Cursor
entirely and calls provider APIs directly. There, model selection is fully under
the user's control via API keys for each provider. The same `config/models.yaml`
format is reused, but slug verification happens against the configured providers
rather than Cursor's whitelist.
