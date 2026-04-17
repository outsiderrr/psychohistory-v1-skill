---
name: psychohistory-character-toolkit
description: "Psychohistory Character Card Generation Toolkit. Orchestrates generation of character cards (personal / organization / collective) and inter-agent relationships, routes to phase-based prompts (prompt-01/02/03/04), enforces the 5 design principles, produces references.md as primary artifact with JSON as compressed index, runs Python schema validation. CLI-first; personal-entity cards prefer the huashu-nuwa Skill when installed, fall back to Research Hand-off otherwise. Triggers: 'generate character card', 'create agent card', 'add agent to scenario', 'make a card for', 'I need a card for', '生成角色卡', '为X做张卡'."
---

# Psychohistory · Character Card Generation Toolkit

> Orchestration skill for generating character cards and inter-agent relationship definitions.
> Delegates phase-by-phase execution to the four phase-based prompts in this directory (`prompt-01` / `prompt-02` / `prompt-03` / `prompt-04`) and enforces cross-cutting concerns (5 design principles, output paths, schema validation).
>
> **CLI-first design**: assumes filesystem access, `git`, `python3` + `jsonschema`, and `WebFetch`. This skill runs inside a CLI agent (Claude Code, Cline, Aider, etc.); it is not designed to execute inside a chat-AI environment.

---

## Environment Assumptions

This skill is designed for **CLI environments** and is **portable across CLI agents** (Claude Code, Cline, Aider, goose, continue.dev, OpenClaw, Manus, etc.). It assumes the following are available:

- **Filesystem read/write** at the project root, discoverable via `git rev-parse --show-toplevel`
- **Python 3** with the `jsonschema` library (or pip-installable: `pip install jsonschema`)
- **git** for project root detection
- **A user with access to a chat AI with search capability** (Perplexity, ChatGPT with Search, Gemini Deep Research, Claude.ai, Kimi, 豆包, 元宝, etc.) — used by the **Research Hand-off** protocol (§Step 3.1 below) during the research phases of card generation. This skill does not call those chat AIs directly; it produces research prompts that the user runs in their preferred tool and brings results back
- **Content policy caveat**: Some chat AIs refuse cognitive modeling of politically sensitive agents. Known case: Gemini refuses to analyze Chinese political leadership (returns empty or "I'm still learning" responses). If your preferred chat AI returns a content-policy refusal, switch to an alternative (Claude.ai and ChatGPT are generally more permissive for analytical/academic framing). As a last resort, generate a `system-derived` card from training knowledge with `strength` capped at `medium` and note the limitation in `honesty_boundaries`

**Intentionally NOT required**: CLI-agent-specific web-fetching tools such as Claude Code's `WebFetch`. Research is delegated to the user's chat AI via copy-paste, which keeps this skill portable across CLI agents.

**Optional enhancement — MCP (Model Context Protocol), not currently implemented**: if your CLI agent supports MCP and you have a search MCP server configured (e.g. Perplexity MCP, Brave Search MCP), the Research Hand-off copy-paste step could in principle be replaced by direct MCP tool calls. This is **not currently developed** — may be added in a future skill version if MCP search servers become widely available and standardized. See `character-toolkit/README.md` §CLI-first Design for the planned MCP integration note.

---

## 5 Design Principles (enforced across all Phases)

All phase-based prompts under this toolkit enforce the following principles. Canonical statement: `character-toolkit/README.md` §方法论原则. Any generated card that violates any of these must return to the responsible Phase for correction — never ship a card that fails these gates.

1. **Two-stage minimum** — research phase first, structuring phase second
2. **references.md is the primary artifact** — JSON is a compressed index compiled from references.md
3. **Evidence strength grading** — `strength: high / medium / low` on every testable claim
4. **Explicit cognitive boundaries** — type-specific disclaimers in `honesty_boundaries`
5. **Historical precedent priority** — cases-backed reasoning ranks above pure theoretical reasoning

---

## Step 0 — Intent Detection

When invoked, classify the user's request into one of two modes:

| Intent pattern | Mode | Example triggers |
|---|---|---|
| "Generate/create a card for X" | **Single-Card** (default) | "Generate IRGC's card", "Create a card for Netanyahu" |
| "Generate multiple cards" / "Set up all the X-side cards" | **Batch** | "Set up all the Iran-side cards for the US-Iran scenario" |

If the user's intent is ambiguous, ask **one** clarifying question before proceeding. Do not guess.

---

## Step 1 — Identify Project Root

Before any file I/O, resolve the project root:

```bash
git rev-parse --show-toplevel
```

Store the result as `<PROJECT_ROOT>`. All subsequent output file paths are computed relative to this root.

If `git rev-parse` fails (not in a git repo), search upward from the current working directory for a `skill/SKILL.md` marker file and use its parent directory as the project root. If neither method works, ask the user for the project root path explicitly.

**Output path convention** (after `<PROJECT_ROOT>` is resolved):

| File | Path |
|---|---|
| Nuwa raw data (personal entities only) | `<PROJECT_ROOT>/skill/characters/nuwa/[agent_id].md` |
| Character references.md (all non-relationship cards) | `<PROJECT_ROOT>/skill/characters/psychohistory/[agent_id].references.md` |
| Character JSON (all non-relationship cards) | `<PROJECT_ROOT>/skill/characters/psychohistory/[agent_id].json` |
| Relationship references.md | `<PROJECT_ROOT>/skill/characters/relationships/[rel_id].references.md` |
| Relationship JSON | `<PROJECT_ROOT>/skill/characters/relationships/[rel_id].json` |

Create any missing subdirectories with `mkdir -p` before writing.

---

## Step 2 — Route by Card Type

From the user's target description, determine the card type:

| Target | Judgment | Prompt file | Output `card_type` |
|---|---|---|---|
| A specific individual | Real person with a name | `prompt-01-personal-entity.md` | `personal-entity` |
| An organization with central decision-making | Has a "final decision-maker" (person or small committee) | `prompt-02-org-entity.md` | `organization-entity` |
| A group without central decision-making | No single decision-maker; behavior is statistical emergence | `prompt-03-collective.md` | `collective` |
| A relationship between two existing cards | User specifies "relationship between A and B" | `prompt-04-relationship.md` | (not a character card) |

**The final decision-maker test**: ask "does this target have a single person or small committee who can make the call?" Yes → entity. No → collective. See `character-toolkit/README.md` §快速判断 for gray-area guidance.

**Gray-zone default**: when ambiguous between `organization-entity` and `collective` (e.g., Iran's Supreme National Security Council — collective mechanism but no single dictator), default to `organization-entity` with `decision_structure.type: "committee"` and explicitly note the peculiarity of the internal mechanism in `honesty_boundaries`.

---

## Step 2.5 — Gather Required Inputs from the Target Prompt's Phase 0

Before invoking the selected prompt's Phase flow, **read its Phase 0 section** using the `Read` tool and identify all required user-supplied inputs. For each required input that was not already provided in the user's initial request, ask the user to provide it. **Do not proceed to Step 3 until all required Phase 0 inputs are collected.**

This intake step exists because `prompt-0X` files have mandatory Phase 0 inputs (scenario background, modeling granularity, endpoint card IDs for relationships, etc.) that the orchestration layer cannot infer from the user's initial intent alone. Without this step, the orchestrator would route to the prompt and let it ask — causing the first Phase to stall on missing parameters.

### Scenario-level setting (applies to all card types)

| Setting | Default | Description |
|---|---|---|
| `output_language` | English | Language for `references.md` content and Phase 3-4 analysis text. **JSON card fields are always English regardless.** Set once per scenario — all cards in the same scenario should use the same output language for consistency. |

If the user is non-English-speaking, proactively ask: *"你希望研究笔记和分析过程用中文还是英文？（JSON 卡的字段始终是英文，不受此设置影响）"* If the user says "Chinese" / "中文", set `output_language = Chinese` for the rest of this scenario.

### Required inputs by card type

| Prompt | User-supplied Phase 0 inputs | Notes |
|---|---|---|
| `prompt-01-personal-entity.md` | target person name (full English), current role, scenario context (optional) | `data_cutoff` is operational — the skill fills it in, not the user |
| `prompt-02-org-entity.md` | target organization name (English), scenario background, modeling granularity (whole organization / specific department), related personal-entity card `agent_id`s (optional) | `current size` is **research output**, goes into Phase 1.6, NOT a Phase 0 input |
| `prompt-03-collective.md` | target group name, `target_agent_id` (which entity agent this collective affects), scenario background, time window | Without a `target_agent_id`, the collective card has no modeling purpose and should be rejected |
| `prompt-04-relationship.md` | `agent_a_id`, `agent_b_id`, scenario background + **verify both endpoint cards exist and pass schema validation** | This is the Phase 0 hard gate; failure terminates the flow immediately |

### Intake procedure

1. Read the selected prompt-0X file and locate its `## Phase 0` section
2. Enumerate each required input. Distinguish between **user-supplied** (needs asking) and **operational** (the skill fills in automatically: `data_cutoff`, date stamps, `created_at`, etc.)
3. Compare against what the user already provided in their initial message
4. **Only ask for inputs that are still missing.** Do not re-ask inputs the user already specified
5. Briefly confirm the full parameter set in one line before proceeding, for example: *"生成组织卡 IRGC，场景：2026-04 美伊停火期即将到期，粒度：整体组织，无关联个人卡"*
6. Then proceed to Step 3

If the user provides a specific target but no scenario background, ask: *"这张卡对应的场景背景是什么？（可以是一句话，比如 '2026-04 美伊停火期即将到期'）"*

If the user provides nothing beyond "generate a card for X", ask the required inputs as a short list, not one-by-one.

---

## Step 3 — Execute the Selected Prompt's Phase Flow

Read the selected prompt file via the `Read` tool and follow its Phase 0 → Phase N sequence exactly as specified. Each phase-based prompt is self-describing; **this skill's role is orchestration, not reimplementation**. The phase content lives in the prompt files.

### 3.1 Research Phase Handling — delegates to Research Hand-off skill

When a Phase requires external research (historical cases, recent decisions, polling data, market movements, current organizational state, interaction history):

**Read `../research-handoff/SKILL.md`** using the Read tool and follow its protocol completely. Pass these parameters from the current context:

- `research_prompt`: load from the current prompt-0X's `## Appendix: Research Hand-off Template`, fill placeholders with target-specific values ({TARGET_NAME}, {TIME_WINDOW_*}, {SCENARIO_CONTEXT}, {TARGET_AGENT_ID}, etc.)
- `output_language`: from Step 2.5
- `integration_target`: the current card's `references.md` file path
- `batch_items`: if in Batch mode (Step 5.3), provide all targets' prompts as a list

The Research Hand-off skill handles: wrapper script detection (`PSYCHOHISTORY_RESEARCH_TOOL`), language instruction prepending, manual copy-paste protocol with chat-AI-specific Canvas/Artifact guidance, paste sanity checking, format tolerance, batch merging, and all error handling.

**The 12-month rule**: for any content within the past 12 months, default to Research Hand-off rather than trusting training knowledge.

When the Research Hand-off skill completes (research integrated into references.md), return to the current prompt-0X's next Phase.

### 3.2 Checkpoint Behavior

Each Phase ends with a Checkpoint that commits the Phase's output to the correct `references.md` section. Behavior differs by interaction mode:

- **Automatic mode** (default): continue directly to the next Phase without pausing
- **Interactive mode**: pause, print the Phase's produced content, and wait for the user to say "continue" or propose adjustments before proceeding

Interactive mode is activated when the user explicitly requests it with "interactive", "step by step", "let me review each phase", "审阅", "一步一步", or similar. If not explicitly requested, run in automatic mode.

### 3.3 Prerequisite Gates (especially prompt-04)

`prompt-04-relationship.md` has a Phase 0 **hard gate**: both endpoint cards must exist and pass schema validation before proceeding. This skill enforces the gate strictly — if either endpoint card is missing or invalid, terminate the flow and report specifically which card is missing. **Do not attempt to "fake" the gate** or generate a relationship card against nonexistent endpoints.

### 3.4 Nuwa Availability Check (prompt-01 only)

`prompt-01-personal-entity.md` Phase 1 delegates cognitive distillation to the `huashu-nuwa` Skill, which is **not built into any CLI agent or chat AI** — it is an external open-source project (github.com/alchaincyf/nuwa-skill) that must be installed via `npx skills add alchaincyf/nuwa-skill`. Before executing prompt-01 Phase 1:

1. **Check availability**: verify `.claude/skills/huashu-nuwa` exists (or equivalent install location for the current CLI agent):
   ```bash
   test -e "$HOME/.claude/skills/huashu-nuwa" && echo "installed" || echo "missing"
   ```
2. **If installed** → proceed with the primary Nuwa path (Phase 1 as written).
3. **If missing** → tell the user: *"prompt-01's primary path requires the `huashu-nuwa` Skill, which is not installed. You have two options: (a) install it with `npx skills add alchaincyf/nuwa-skill --yes` then restart this session, or (b) proceed now using the Appendix Research Hand-off Template as a fallback — the card will be generated via single-prompt research rather than Nuwa's 6-agent parallel distillation, which is acceptable for information-rich public figures but may produce less complete results for information-scarce subjects."* Wait for the user's decision before proceeding. If fallback is chosen, follow the Appendix instead of Phase 1, and record the fallback in references.md §1.

**Do not silently fall back.** A silent fallback was the root cause of dry-run Finding F-022 — the user thought they were getting the primary path when in fact the skill was skipping Nuwa entirely.

### 3.5 Writing Output Files

As each Phase completes, write its output to the correct file via the `Edit` or `Write` tool:

- Phase 1-N research notes → append to `references.md`
- Final Phase JSON compilation → write to `.json`

Use the paths computed in Step 1. Always write to the canonical path — do not write to temp locations and move.

---

## Step 4 — Schema Validation

**Prerequisite (one-time setup)**: the `jsonschema` Python package must be installed. If not already available:

```bash
# Option 1 (recommended): project-level virtual environment — safest on all platforms
python3 -m venv .venv && source .venv/bin/activate && pip install jsonschema

# Option 2: global install (works if your system Python allows it)
pip install jsonschema

# Option 3: if blocked by PEP 668 (Python 3.12+ on macOS / managed Linux distros)
pip install --break-system-packages jsonschema
```

**Note on PEP 668**: Python 3.12+ on macOS (Homebrew) and many Linux distros marks the system Python as "externally managed", blocking both `pip install` and `pip install --user`. Option 1 (venv) avoids this entirely. Option 3 overrides the protection — safe for a single package like jsonschema but not recommended as a general habit.

The skill does **not** auto-install dependencies. If `jsonschema` is missing when Step 4 runs, the validation script (below) gracefully falls back: it prints an install instruction, marks the card as "saved but not schema-validated", and exits 0. The user can install `jsonschema` later and manually re-run validation with the same Python block. This fallback is documented in §Failure Modes.

After the selected prompt's final Phase produces JSON, run JSON Schema validation against `skill/references/character-schema.md`:

```bash
python3 << 'PYEOF'
import json, re, sys

PROJECT_ROOT = "<RESOLVED_PROJECT_ROOT>"
SCHEMA_MD = f"{PROJECT_ROOT}/skill/references/character-schema.md"
CARD_JSON = "<OUTPUT_CARD_PATH>"

with open(SCHEMA_MD) as f:
    content = f.read()

section_start = content.find("## 2. JSON Schema Definition")
if section_start < 0:
    sys.exit("⚠️  Could not locate JSON Schema section in character-schema.md")

rest = content[section_start:]
match = re.search(r"```json\n(\{[\s\S]*?\n\})\n```", rest)
if not match:
    sys.exit("⚠️  Could not extract JSON Schema block")

schema = json.loads(match.group(1))

with open(CARD_JSON) as f:
    card = json.load(f)

try:
    import jsonschema
except ImportError:
    print("⚠️  jsonschema not installed. Install with: pip install jsonschema")
    print(f"Card has been saved to {CARD_JSON} but not schema-validated.")
    print("Manual validation required.")
    sys.exit(0)

try:
    jsonschema.validate(card, schema)
    print(f"✓ Card passes schema validation: {CARD_JSON}")
except jsonschema.ValidationError as e:
    print(f"✗ Schema validation failed at {'/'.join(str(p) for p in e.path)}:")
    print(f"  {e.message}")
    sys.exit(1)
PYEOF
```

Before running: substitute `<RESOLVED_PROJECT_ROOT>` with the actual project root path from Step 1, and `<OUTPUT_CARD_PATH>` with the actual JSON file path from Step 3.

If validation fails, report the specific field and error, return to the responsible Phase (usually the JSON compilation Phase) for correction, and re-run validation.

**Relationship cards**: `character-schema.md` does not cover relationships (they live in `schema.md §3`). For relationships, run a reduced manual check against the self-check list in `prompt-04-relationship.md` Phase 6 instead of the Python validation above.

---

## Step 5 — Batch Coordination (when in Batch Mode)

Batch mode handles multiple related cards in dependency order.

### 5.1 Parse the batch request

The user's request typically looks like:

> "Set up the Iran-side cards for the US-Iran 2026 scenario: I need iran-government, irgc-leadership, plus the iran-gov ↔ irgc relationship."

Parse this into two lists:

- **Character cards**: `iran-government` (organization-entity), `irgc-leadership` (organization-entity)
- **Relationships**: `rel-001 iran-gov ↔ irgc` (loyalty)

### 5.2 Sort by dependency order

Rules:

1. All character cards first — they have no dependencies on other cards
2. Relationships last — their Phase 0 gate requires endpoint cards to exist
3. Within character cards, order is flexible

### 5.3 Batch Research Collection (reduces N copy-pastes to 1)

Before executing individual card generation, collect research for all **non-relationship cards** in one merged Research Hand-off call. This reduces N copy-pastes to 1 for users without a wrapper script, and collapses per-card API calls to one batch call for wrapper-script users.

Relationships are handled separately in step 5.5 — they need endpoint character cards to exist first.

**Procedure**:

1. **Gather research needs**: for each character card in the batch (personal / organization / collective), load its corresponding prompt-0X's `## Appendix: Research Hand-off Template`.

2. **Fill each template** with that card's parameters (`{TARGET_NAME}`, `{TIME_WINDOW_*}`, `{SCENARIO_CONTEXT}`, etc.).

3. **Construct a merged research prompt** using clear delimiters so the response can be split per target:

~~~
You will research multiple targets for a Psychohistory scenario. For EACH
target listed below, return findings as a separate clearly-delimited section.

=== RESPONSE FORMAT ===

Use the following exact delimiters for each target's section. Do not add
prose outside these delimited sections.

    <<<TARGET: [target_id]>>>
    [complete research sections §1-§N as specified in the target's template]
    <<<END TARGET: [target_id]>>>

=== TARGETS ===

--- Target 1: [target_id_1] ([card_type_1]) ---

[filled-in Research Hand-off Template for target 1]

--- Target 2: [target_id_2] ([card_type_2]) ---

[filled-in Research Hand-off Template for target 2]

... (continue for all character cards in the batch)

=== END TARGETS ===
~~~

4. **Dispatch the merged prompt**:
   - **If `PSYCHOHISTORY_RESEARCH_TOOL` is configured** (see §Step 3.1.0): pipe the entire merged prompt to the wrapper via stdin, capture the response from stdout — one API call for the whole batch
   - **Otherwise**: emit the merged prompt to the user as one copy-pasteable block (preceded by the chat-AI recommendation from §Step 3.1.1), wait for the user to paste the response back — one copy-paste round-trip for the whole batch

5. **Parse the returned content**: split the response on `<<<TARGET: [id]>>>` and `<<<END TARGET: [id]>>>` delimiters to get per-target research. Apply the format tolerance protocol from §Step 3.1.1 — the target boundaries must be identifiable, but minor variations inside each section are acceptable.

6. **Cache per-target research**: store each target's parsed research keyed by `agent_id` for consumption by step 5.4.

**Batch size guidance**:

- **Without wrapper (manual paste)**: recommend **max 4 character cards** per merged prompt to avoid exceeding chat AI context limits (Perplexity, ChatGPT Search, etc. typically cap input around 30-50K tokens). If the user requests more, warn and split into sub-batches of ≤4.
- **With wrapper**: constrained by the provider's API context window (modern LLMs: 100K+ tokens — 8+ cards comfortable). Quality may still degrade past ~6 cards per call; consider sub-batching for very large sets.

### 5.4 Execute individual card generation

For each character card in dependency-sorted order:

1. Run Steps 1-4 (single-card flow) for that card, **but skip Step 3.1 Research Phase Handling** — use the cached per-target research from step 5.3 instead
2. Integrate the cached research into the card's `references.md` at the appropriate `§1`-`§N` sections (per the prompt-0X's phase structure)
3. Continue with the non-research phases (structural analysis, pattern extraction, JSON compilation, Python schema validation)
4. Collect the result (paths, validation status)
5. If validation fails and cannot be resolved, **stop the batch** and report the failing card

### 5.5 Handle relationship cards

If the batch includes relationship cards, handle them **after** all character cards are complete so that `prompt-04`'s Phase 0 endpoint-card gate passes.

- **1-2 relationships**: run Steps 1-4 individually for each (standard single-card flow, including Step 3.1 Research Phase Handling for interaction history)
- **3+ relationships**: optionally run a second merged research collection using `prompt-04-relationship.md`'s Research Hand-off Template — collect all interaction histories in one call, then generate individual relationship cards from the cached research

### 5.6 Report summary

After all items complete (or after stopping on failure):

```
Batch generation summary
========================
Character cards generated:
  ✓ iran-government      (organization-entity, passed validation)
  ✓ irgc-leadership      (organization-entity, passed validation)

Relationships defined:
  ✓ rel-001  iran-government ↔ irgc-leadership  (loyalty, current_value: 70, loyal mode)

Files written:
  <PROJECT_ROOT>/skill/characters/psychohistory/iran-government.{json,references.md}
  <PROJECT_ROOT>/skill/characters/psychohistory/irgc-leadership.{json,references.md}
  <PROJECT_ROOT>/skill/characters/relationships/rel-001.{json,references.md}

Research mode: [wrapper: perplexity.sh] or [manual Hand-off]
Research calls: 1 merged (character cards) + 1 merged (relationships) = 2 total
  (vs. 3 individual calls in the non-batched flow)

Total: 3 artifacts, 0 validation failures
```

---

## Interaction with `skill/SKILL.md`

`skill/SKILL.md` is the main Psychohistory scenario analysis skill. Its Phase 1.5 "Character Card Loading Router" handles the case when a scenario analysis needs an agent card that isn't in the official library.

Under the v1.1 architecture:

- **Phase 1.5 Tier 1** (official library lookup) — unchanged
- **Phase 1.5 Tier 2** (CLI auto-generation) — **delegates to this toolkit**. The main scenario skill invokes `psychohistory-character-toolkit` with the target agent and scenario context; this skill runs the full phase-based pipeline and saves the card. For personal-entity cards, Phase 1 prefers the `huashu-nuwa` Skill when installed and falls back to the Research Hand-off path inside `prompt-01` when it is not

This replaces the pre-v1.1 Tier 2 which directly invoked Nuwa. Nuwa is still invoked, but now as a sub-step inside `prompt-01-personal-entity.md` Phase 1, not directly from the main scenario skill.

---

## Failure Modes & Responses

| Failure | Response |
|---|---|
| `git rev-parse --show-toplevel` fails | Search for `skill/SKILL.md` marker file; if not found, ask user for project root |
| Target card type unclear | Ask one clarifying question; default to `personal-entity` if the user's description mentions a named individual |
| prompt-04 Phase 0 gate fails (missing endpoint cards) | Report specifically which endpoint is missing; offer to generate it first (sub-invocation) or terminate |
| Nuwa.skill not installed (CLI, prompt-01 Phase 1) | Follow Step 3.4: inform user Nuwa is external (`npx skills add alchaincyf/nuwa-skill`), ask them to (a) install + restart, or (b) proceed with Research Hand-off fallback. Never fall back silently. |
| Nuwa.skill invocation fails mid-run (CLI, prompt-01 Phase 1) | Report Nuwa's error; offer to retry, fall back to Research Hand-off with fallback noted in references.md §1, or abort |
| Recent-data research cannot be completed via Research Hand-off | Pause, show the user what was attempted, offer to (a) retry with a different chat AI, (b) fall back to training knowledge with `strength: low`, or (c) mark fields as "data unavailable"; never fabricate |
| JSON Schema validation fails | Report the specific field and error; return to the compilation Phase for correction; re-run validation |
| Python `jsonschema` library missing | Report and instruct `pip install jsonschema`; card is still saved but unvalidated (flagged) |
| Chat AI refuses research due to content policy (e.g., Gemini on Chinese/Russian/Iranian leadership) | Switch to alternative chat AI (Claude.ai, ChatGPT); if all refuse, fall back to `system-derived` card with `strength` capped at `medium` and note limitation in `honesty_boundaries` |
| File write fails | Report path + error; ask user to check permissions |

---

## Quick Reference

| Action | How to invoke |
|---|---|
| Generate a personal entity card | "Generate a card for [person name]" |
| Generate an organization card | "Create a card for [organization name]" |
| Generate a collective card | "Create a profile for [group name] affecting [target_agent]" |
| Define a relationship | "Define the relationship between [agent_a] and [agent_b] in [scenario]" |
| Batch generation | "Generate all the [X-side] cards for [scenario]" |
| Interactive mode | Add "step by step" or "interactive" to any of the above |

---

## Reference Documents

- `prompt-01-personal-entity.md` — phase-based flow for personal entities (Phases 0-4)
- `prompt-02-org-entity.md` — phase-based flow for organization entities (Phases 0-6)
- `prompt-03-collective.md` — phase-based flow for collective agents (Phases 0-8)
- `prompt-04-relationship.md` — phase-based flow for inter-agent relationships (Phases 0-6)
- `character-toolkit/README.md` — toolkit index, 5 design principles, save paths, CLI-first statement
- `../references/character-schema.md` — polymorphic v1.1 schema for all three card types
- `../references/schema.md` — scenario schema including relationship structure
- `../SKILL.md` — main Psychohistory scenario analysis skill (Phase 1.5 delegates to this toolkit)
