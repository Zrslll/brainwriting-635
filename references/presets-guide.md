# Presets Guide

This document explains when to use each Phase 1 preset and how the orchestrator
should select between them.

## Available Presets (v0.1)

| Preset | Roles | Best For |
|---|---|---|
| `general` | Pragmatist, Contrarian, UX Advocate, Engineer, Business, Visionary | Non-IT topics; lifestyle, planning, content, decisions, anything without a clear technical domain |
| `it-product` | Product Manager, UX/UI Designer, Frontend Engineer, Backend Engineer, QA/Edge Cases, Growth/Analytics | Software product features, UX work, API design, anything user-facing software |

## Selection Logic

The orchestrator should auto-select a preset based on the task statement, then
confirm with the user. Logic:

1. **Scan the task** for keywords matching `triggers.keywords_en` or
   `triggers.keywords_ru` from `config/presets/it-product.yaml`
2. **If 2+ IT triggers match**, default to `it-product`
3. **If 0-1 triggers match**, default to `general`
4. **Always show the choice to the user** with reasoning, e.g.:
   > "I detected this is an IT product task (matched: 'feature', 'API', 'users
   > of the app'). I'll use the it-product preset. Override with 'general' if
   > you prefer."

## When to Override the Auto-Selection

### Use `general` for an IT-sounding task when:

- The task is about **adoption, communication, or organizational change**
  rather than building (e.g. "How do we get the team to write more tests?")
- The task is about **business strategy** with software as a backdrop
  (e.g. "Should we pivot from B2B to B2C?")
- The task is about **hiring, team structure, or process** rather than product

### Use `it-product` for a non-IT-sounding task when:

- The task uses informal language but is fundamentally about software
  (e.g. "Make our checkout less annoying" - that's frontend + UX + analytics)
- The output will be implemented as software regardless of how it's phrased
- You explicitly want IT roles' lenses on a borderline case

## Why So Few Presets in v0.1

Two reasons:

1. **Prove the methodology first.** Adding presets is cheap. Proving that
   the two-phase 6-3-5 + Six Hats workflow produces useful output is the
   actual MVP risk. More presets without methodology validation just creates
   maintenance burden.

2. **Two presets cover ~80% of expected use.** Most brainwriting in this
   project context will be either personal/general decisions or IT product
   work. Specialized presets (marketing, content strategy, architecture
   deep-dive, hiring) can be added as separate files in
   `config/presets/` once we know which ones are actually requested.

## Adding a New Preset (Future)

Pattern for adding `config/presets/<new-preset>.yaml`:

1. **Pick 6 roles** with genuinely distinct lenses. If two roles would say the
   same thing 80% of the time, you have 5 effective roles, not 6.
2. **Write each role's lens** as 2-3 sentences focused on what they uniquely
   notice that others miss.
3. **List 3 typical questions** per role that capture their thinking pattern.
4. **List 2-3 things to avoid** per role - this prevents drift into other
   roles' territory.
5. **Optionally add `triggers`** with keyword arrays to enable auto-selection.
6. **Test on 2-3 representative tasks** before declaring the preset done.

### Anti-Patterns When Designing Roles

- **Roles that are obviously redundant.** "Backend Engineer" and "DevOps" both
  think about systems; pick one or differentiate sharply (e.g. "Backend"
  thinks data and APIs, "SRE" thinks operations and incidents).
- **Roles that are too narrow.** "iOS Developer" generates only iOS-specific
  ideas. Use "Mobile Engineer" or "Frontend Engineer" instead.
- **Roles that are demographic, not functional.** "Senior" vs "Junior" engineer
  is not a lens; it's an experience level. Use functional roles.
- **Including a Critic role.** This duplicates the Black hat's job in Phase 2.
  Phase 1 is for divergence; Phase 2 has structured critique. The Contrarian
  in `general` is borderline - it's kept because contrarian thinking generates
  IDEAS (e.g. "what if we did the opposite") rather than just critiques.

## Future Preset Candidates

Likely additions based on common needs:

- **`it-architecture`** - Tech Lead, Backend, Frontend, DBA, Security, SRE
  (for architecture decisions, not feature design)
- **`marketing`** - Brand, Performance, Content, Community, Analyst, Strategist
- **`feature-discovery`** - User Researcher, PM, Designer, Engineer, Data, Sales
  (for finding what to build, before designing it)
- **`content-strategy`** - Editor, SEO, Writer, Designer, Audience, Distribution

These are not implemented in v0.1. Add them when there is a concrete need.
