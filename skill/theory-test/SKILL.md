---
name: theory-test
description: "Theory Test: hypothesis validation engine for testing a specific theory about an agent's decision-making model. Input a theory as a cognitive framework perturbation, output a structured comparison between the standard and alternative frameworks against historical events. Uses three-engine analysis for dual-run validation and research-handoff for gathering test events. Triggers: 'test this theory', 'validate my hypothesis', 'is this theory correct', 'compare frameworks', 'what if their real goal is', 'test alternative explanation'."
---

# Theory Test — Hypothesis Validation Engine

> Given a specific theory about an agent's decision-making model, test it against observed facts. Run historical events through both the standard framework and the alternative framework, compare explanatory power, and identify future events that would discriminate between them.
>
> Independent atomic skill. Uses `three-engine` for dual-run validation and `research-handoff` for gathering test events via Method A references.

---

## When to Use

When the user has a **specific hypothesis** about an agent's decision-making model and wants to test it systematically:

- "I think the Fed's real goal is protecting asset prices, not controlling inflation"
- "I think Trump is being influenced by Thiel through Vance more than it appears"
- "I think Iran's moderate faction is actually stronger than the public posture suggests"
- "I think this company's 'restructuring' is actually preparation for an acquisition"

**This is NOT for**:
- Explaining observed events without a pre-existing theory -> Use `news-interpreter`
- Forward scenario analysis -> Use the main Psychohistory skill
- Generating character cards -> Use `character-toolkit`

**Key distinction**: theory-test starts from a **user-provided hypothesis** and checks it against facts. news-interpreter starts from **observations** and generates hypotheses. news-interpreter calls the same validation logic internally for each candidate it generates.

---

## Inputs

| Input | Required | Description |
|---|---|---|
| Target agent | Yes | Which agent's model is being challenged. Must have a character card in `characters/psychohistory/` (or be a very well-known public figure — see Prerequisites). |
| Theory statement | Yes | The user's hypothesis in natural language. Translated into a framework perturbation in Phase 2. |
| Test events | No | Specific historical events to test against. If not provided, gathered via Research Hand-off in Phase 3. |
| Time window | No | How far back to look for test events. Default: 12 months. |
| output_language | No | Language for analysis output. Default: English. |

---

## Prerequisites

### Character Card

This skill **requires** the target agent's character card to define the "standard framework" baseline.

- **If card exists** in `characters/psychohistory/{agent_id}.json` -> load it
- **If card does not exist** -> inform the user: *"Theory-test requires a baseline character card. Generate one for [agent] using `character-toolkit` first."*
  - Exception: for very well-known public figures, offer to proceed with a `system-derived` baseline — but warn that the comparison is shallower (both frameworks are approximations)

### Existing alternative_frameworks

If the target card already has an `alternative_frameworks` entry matching the user's theory -> use it directly (skip Phase 2). Inform the user: *"This theory already exists as alternative framework [id] in [agent]'s card. Running validation."*

---

## Execution Flow

### Phase 0: Understand the Theory

Confirm with the user:

1. **Which agent?** Whose decision-making model does this theory challenge?
2. **What does the theory claim?** One sentence: what is different from the standard model?
3. **What level of change?**
   - Core objectives (what the agent wants)
   - Mental models (how the agent sees the world)
   - Decision heuristics (how the agent makes choices)
   - Hidden influences (who is really driving decisions)
   - A combination
4. **What triggered this theory?** What did the user observe? (These become mandatory test events.)

#### Agent Typing Check

After identifying the target agent and understanding the theory, verify that the agent's card type matches the theory's implications.

**Personal vs. Organization typing — divergence test**: In this specific scenario, can the agent's personal interest point toward a *different decision direction* than organizational/national interest? If yes → personal entity card. If no → organization/leadership card. A leader's strong personal control over an organization is NOT itself a reason to use personal entity modeling — only actual directional divergence matters.

> Example (no divergence → organization): Xi Jinping in a dense diplomacy scenario. Personal interest = demonstrate China is not isolated. CCP institutional interest = demonstrate China is not isolated. Interests align → organization entity.

**Proactive reframing**: Users often focus on *what* they think is happening but may not have considered *at what level* it operates. If the theory implies personal-institutional divergence (e.g., "Country X's real strategy is...") but the divergence test shows interests align, flag this: the theory might be about institutional dynamics, not personal motives. Conversely, a theory framed as institutional may actually hinge on a leader's personal incentives diverging from organizational interest. **Suggest reframing the theory when the divergence test reveals a better-fitting target** — this sharpens the theory before testing, not after.

If the user provides a clear theory with context -> proceed to Phase 1.

---

### Phase 1: Load the Standard Framework

1. Load the target card from `characters/psychohistory/{agent_id}.json`
2. Validate against `references/character-schema.md`
3. Extract the baseline:
   - `mental_models` (standard set)
   - `decision_heuristics` (standard set)
   - `core_objectives` and `priority_ranking`
   - `concession_triggers` and `red_lines`
4. Card freshness check: if `data_cutoff` > 6 months before relevant events, offer incremental Research Hand-off via `../research-handoff/SKILL.md`

Report the standard framework summary to the user. Confirm it matches their understanding of the "conventional view" before proceeding.

---

### Phase 2: Construct the Alternative Framework

Translate the user's natural-language theory into a **concrete framework perturbation**.

#### 2.1 Identify What Changes

| Theory type | Card fields affected |
|---|---|
| "Their real goal is X" | `core_objectives`, `priority_ranking` |
| "They see the world as X" | `mental_models` (replace or add entries) |
| "They always do X because Y" | `decision_heuristics` (replace or add) |
| "They're controlled by Z" | `concession_triggers` anchored on Z; add hidden mental_model |
| "They won't do X" | `red_lines` |

#### 2.2 Construct the Alternative

For each affected field, specify the alternative version:

```
## Alternative Framework: [Theory name]

### Standard says:
- mental_models: [current from card]
- decision_heuristics: [current]
- core_objectives: [current]

### Theory says instead:
- mental_models: [modified — which replaced/added/removed]
- decision_heuristics: [modified]
- core_objectives: [modified if applicable]

### Key behavioral predictions that differ:
1. Under standard, agent would [X] in situation [S]
2. Under theory, agent would [Y] in situation [S]
```

#### 2.3 User Confirmation

Present the constructed alternative to the user: *"Does this correctly capture your theory?"*

This step is critical — a mis-specified alternative framework produces meaningless results.

---

### Phase 3: Gather Test Events

Test events must be **actual events that already happened** — not hypotheticals.

#### 3.1 Sources (priority order)

1. **Events the user provided** in Phase 0 (mandatory)
2. **Events from the agent's character card** (key_decisions or historical data)
3. **Events gathered via Research Hand-off** — read `../research-handoff/SKILL.md` and follow its protocol

#### 3.2 Research Prompt Template

```
I need recent decision events for [agent name] to test a hypothesis.

Find [5-8] significant decisions or actions by [agent name] in the past [time_window]:

For each event:
1. Date and description
2. Context (what prompted the decision)
3. Alternatives available (what else could they have done)
4. Stated rationale (what they said about why)
5. Outcome

Focus on events with genuine choice — avoid routine/ceremonial.
Prioritize:
- Surprising or debated decisions (unclear motivation)
- High-stakes (reveals priorities)
- Recent

Return organized by event, sources cited.
```

#### 3.3 Minimum Event Set

**>= 5 test events** for meaningful comparison. Fewer -> warn and proceed with caveat.

---

### Phase 4: Dual-Run Three-Engine Analysis

The core validation step. **Read `../three-engine/SKILL.md` and apply its [GT] / [PSY] / [ORG] framework** — twice.

#### 4.1 Run A: Standard Framework

For each test event, three-engine analysis:
> "Given agent [X]'s standard cognitive framework, would they have made this decision in this context?"

[PSY] uses **standard** mental_models, decision_heuristics, known_biases from card.

Record per event: explained / contradicted / neutral.

#### 4.2 Run B: Alternative Framework

Same events, same question, but with the **modified** framework from Phase 2.

[PSY] uses **alternative** mental_models and decision_heuristics.

Record per event: explained / contradicted / neutral.

#### 4.3 Comparison Matrix

```
| Event | Standard Framework | Alternative Framework | Winner |
|---|---|---|---|
| Event 1 | explained [PSY: mm-01] | explained [PSY: alt-mm-02] | tie |
| Event 2 | contradicted [GT] | explained [PSY: alt-obj-01] | alternative |
| Event 3 | explained [ORG: inertia] | neutral | standard |
| ... | ... | ... | ... |
| TOTAL | N explained, M contradicted | P explained, Q contradicted | [overall] |
```

---

### Phase 5: Compare and Output

#### 5.1 Overall Assessment

One of four verdicts:

1. **Alternative clearly stronger**: explains more events with fewer contradictions. The theory has significant merit.
2. **Roughly equal**: both explain similar numbers. Data doesn't discriminate — more events needed.
3. **Standard clearly stronger**: theory contradicts more events than it explains. Likely wrong as stated (but may contain partial insight).
4. **Inconclusive**: too few events, or frameworks predict similar outcomes for most events.

#### 5.2 Discriminating Future Events

**The most actionable output.** What future events would decisively favor one framework?

Per discriminator:
```
**Discriminating Event [N]**: [what could happen]
- Standard predicts: [X]
- Alternative predicts: [Y]
- Time window: [when observable]
- Observability: [how to check — public data? News? Reports?]
```

Aim for 3-5 discriminators prioritizing:
- Short time windows (testable soon)
- High observability (publicly verifiable)
- Maximum divergence (frameworks predict opposite outcomes)

#### 5.3 Partial Insights

Even if the overall theory fails, note events where the alternative was clearly better. These may point to a refined theory.

#### 5.4 Output Format

```
# Theory Test: [Theory title]

## Theory Under Test
**Agent**: [name]
**Claim**: [one sentence]
**Framework perturbation**: [summary]

## Test Events ([N] events, [time window])
1. [Event] — [date]
2. ...

## Comparison Results

| Event | Standard | Alternative | Winner |
|---|---|---|---|
| ... | ... | ... | ... |

**Standard**: [N] explained, [M] contradicted, [K] neutral
**Alternative**: [P] explained, [Q] contradicted, [R] neutral

## Assessment: [clearly stronger / roughly equal / clearly weaker / inconclusive]

[Detailed reasoning with [GT]/[PSY]/[ORG] tags]

## Discriminating Events (What to Watch)
1. [Discriminator 1] — [timeframe]
2. [Discriminator 2] — [timeframe]
3. [Discriminator 3] — [timeframe]

## Partial Insights
[Events where alternative was notably better, even if overall weaker]

## Epistemic Boundaries
- This is a structured comparison, not a proof. "Clearly stronger" means
  better fit to observed data — not "true."
- Bounded by the [N] events examined. Different events might yield
  different results.
- Both frameworks are simplifications of real decision-making.
[Additional caveats as applicable]
```

---

### Phase 6: Interaction (Ongoing)

#### 6.1 Refine the Theory
User: "What if I adjust my theory to also include X?"
-> Update the alternative framework, re-run Phase 4 with modified version, update assessment.

#### 6.2 Add Test Events
User: "Here's another event — does it change the result?"
-> Add to event set, re-run comparison for the new event, update overall assessment.

#### 6.3 Flip Perspective
User: "Test the opposite — maybe the standard framework is wrong"
-> Swap roles: user's theory becomes "standard," current standard becomes "alternative." Same comparison. Guards against anchoring on the card's baseline.

#### 6.4 Upgrade to Card
User: "The theory seems right — update the card"
-> If assessed as "clearly stronger," offer to:
  - Add as `alternative_frameworks` entry (conservative)
  - Replace the standard framework (aggressive — only with strong evidence)
Tag update `source: theory-test-validated`.

---

## Quality Standards

| Good Analysis | Bad Analysis |
|---|---|
| Both frameworks tested against ALL events | Cherry-picks events favoring the theory |
| Comparison matrix with per-event results | Vague "theory seems plausible" |
| Discriminating future events identified | No "what to watch" output |
| [GT]/[PSY]/[ORG] tags on reasoning | Undifferentiated narrative |
| Epistemic boundaries displayed | Assessment presented as proof |
| Partial insights noted when theory fails | Binary "works / doesn't work" |
| Standard framework at full strength | Standard weakened to favor the theory |

---

## Never Do

- **Never confirm a theory without dual-run comparison** — even if it "sounds right," run the test
- **Never straw-man the standard framework** — present it at full strength
- **Never skip events that contradict the user's theory** — those are the most informative
- **Never output precise probability numbers** — assessment categories only
- **Never test without a character card baseline** — the comparison needs a concrete standard
- **Never treat "roughly equal" as confirmation** — it means data doesn't discriminate
- **Never fabricate test events** — only actual historical events
