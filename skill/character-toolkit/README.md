# Psychohistory · Character Card Generation Toolkit (Index)

**English** · [中文](./README_CN.md)

> This toolkit contains standardized prompts for generating all types of character cards used by Psychohistory.
> Pick the prompt file that matches the type of character you want to generate.

---

## Character Type Router

```
What are you generating?
│
├── A specific individual? (Trump, Netanyahu, Powell)
│   └── 👉 Use prompt-01-personal-entity.md (invokes the Nuwa skill)
│
├── An organization? (Federal Reserve, IRGC, Apple Inc., Iranian government)
│   └── 👉 Use prompt-02-org-entity.md
│
├── A collective? (MAGA voters, US stock market participants, Iranian public)
│   └── 👉 Use prompt-03-collective.md
│
└── A relationship between agents? (Trump ↔ Vance, Iran gov ↔ IRGC)
    └── 👉 Use prompt-04-relationship.md
```

---

## File List

| File | Purpose | Output Format |
|------|---------|---------------|
| `prompt-01-personal-entity.md` | Personal entity card (invokes Nuwa skill) | `[agent_id].json` |
| `prompt-02-org-entity.md` | Organization entity card (government / company / military) | `[agent_id].json` |
| `prompt-03-collective.md` | Collective agent profile | `[agent_id].json` |
| `prompt-04-relationship.md` | Inter-agent relationship definition | `[rel_id].json` |

---

## CLI-first Design

This toolkit **defaults to CLI environments** (e.g. Claude Code). All phase-based prompts and `SKILL.md` assume:

- Filesystem read/write access
- `git rev-parse --show-toplevel` available for automatic project root detection
- `python3` + `jsonschema` available for automated schema validation
- `WebFetch` available for retrieving source data during research phases

**Chat-AI as an escape hatch**: if you want to generate a card using ChatGPT, Claude.ai, Trae, Gemini, or similar chat AIs, do not copy `prompt-0X` files directly — they are written for CLI. Instead, invoke `SKILL.md` in **Export Mode**: from your CLI session, tell it the target and scenario, and it will emit a self-contained, chat-AI-adapted prompt you can paste into your chat AI. After the chat AI responds, bring the references.md + JSON content back to your CLI session so this toolkit can validate and save them.

This design lets every CLI step rely confidently on filesystem, Python validation, WebFetch, and other tools, rather than compromising for chat-AI compatibility. Chat-AI support is handled via Export Mode as a single point of adaptation.

---

## Universal Rules

These rules apply to every type of character card:

1. **All JSON must pass standard parser validation** — no syntax errors allowed
2. **All text content in English** — fields like `name` and `description` must be English-only
3. **Date format: YYYY-MM-DD only** — no incomplete dates accepted
4. **`agent_id` is lowercase alphanumeric + hyphens** — e.g. `us-federal-reserve`, `maga-base`
5. **Every card must have `honesty_boundaries`** — explicitly stating what the card cannot capture

---

## Methodology Principles (v1.1)

The following 5 principles apply across all **phase-based prompts** (prompt-02 / 03 / 04) and form the foundation of the character-card generation methodology. prompt-01 (personal entity) inherits principles 1, 3, 4, and 5 through its dependency on the Nuwa multi-phase distillation pipeline; principle 2 applies to it as well.

1. **Two-stage minimum** — Research phase first, structuring phase second. Never go directly from blank to JSON.
2. **references.md is the primary artifact** — JSON is its compressed index. Write the notes first → compile the JSON from them → verify every JSON field points back to specific evidence in references.md.
3. **Evidence strength grading** — Every conclusion is annotated `strength: high / medium / low`. `high` requires multi-case support; `low` is allowed but must be explicitly flagged.
4. **Explicit cognitive boundaries** — `honesty_boundaries` must include type-specific declarations:
   - Organizations: factional divergence disclaimer
   - Collectives: internal heterogeneity disclaimer
   - Relationships: scenario-binding disclaimer
5. **Historical precedent priority** — Reasoning backed by historical cases ranks above pure theoretical reasoning; precedent-free inferences are flagged `strength: low`.

These 5 are **hard requirements**, not suggestions. Any JSON produced by a phase-based prompt that violates any of them must go back to the corresponding Phase to add evidence or correct its strength grading.

---

## Save Paths

| Type | Path |
|------|------|
| Nuwa raw data | `characters/nuwa/[agent_id].md` |
| Personal entity JSON | `characters/psychohistory/[agent_id].json` |
| Organization entity JSON | `characters/psychohistory/[agent_id].json` |
| Collective agent JSON | `characters/psychohistory/[agent_id].json` |
| Relationship JSON | `characters/relationships/[rel_id].json` |
| References file (evidence chain) | Same directory as the JSON, with `.references.md` suffix |

Every character card and every relationship definition must also produce a references file (`.references.md`) documenting the evidence chain behind each conclusion. The references file is for users who want to verify or refine the conclusions; it does not affect engine execution.

---

## Quick Judgment: Entity or Collective?

Ask yourself one question: **Does this character have a single "final decision-maker"?**

- Yes → Entity. Even large organizations qualify as entities if one person or a small committee can make the call.
- No → Collective. No one can represent the whole group in making a decision; behavior is a statistical emergent outcome.

**What about gray zones?**

Some characters fall between the two — e.g. Iran's Supreme National Security Council, which has a collective decision-making mechanism but no single dictator. In such cases, model it as an **organization entity**, but explicitly note the peculiarities of the internal decision-making mechanism in the card.
