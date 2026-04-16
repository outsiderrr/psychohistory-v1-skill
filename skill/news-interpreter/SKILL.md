---
name: news-interpreter
description: "News Interpreter: abductive inference engine for analyzing observed events. Input a set of related events (>=3), output ranked candidate theories explaining the pattern. Each theory is expressed as a cognitive framework perturbation on relevant agents. Uses three-engine analysis for validation, character cards for agent modeling, and research-handoff for fact verification. Triggers: 'interpret this news', 'why did this happen', 'what does this pattern mean', 'explain these events', 'abductive inference', 'reverse reasoning from events', 'what logic connects these'."
---

# News Interpreter — Abductive Inference from Observed Events

> Given a set of related events, generate and rank candidate theories that explain the underlying pattern. Each theory is a structured perturbation on one or more agents' cognitive frameworks — not a vague narrative, but a testable proposition.
>
> Independent atomic skill. Uses `three-engine` for validation and `research-handoff` for fact gathering via Method A references.

---

## When to Use

When you observe events that **lack a unified explanation** and want to understand the decision-making models behind them:

- A cluster of diplomatic events: "Four countries visited China this week — what strategic logic connects them?"
- An unexpected policy shift: "Pakistan military deployed to Saudi air base — what does this reveal about both sides' models?"
- A pattern across seemingly unrelated actions: "These three trade decisions look contradictory — what underlying framework makes them consistent?"

**This is NOT for**:
- Forward scenario analysis ("what will happen next?") -> Use the main Psychohistory skill
- Testing a specific user-provided theory -> Use `theory-test`
- Generating character cards -> Use `character-toolkit`

**Key distinction**: news-interpreter starts from **observations** and works backward to **models**. Psychohistory starts from **models** and works forward to **predictions**.

---

## Inputs

| Input | Required | Description |
|---|---|---|
| Observed events | Yes | A set of >= 3 related events to analyze. Each event: what happened, when, who was involved. More events = stronger inference. |
| Focus question | No | What specific aspect the user wants explained. Default: "What decision-making model best explains this pattern?" |
| Agent scope | No | Which agents to focus on. Default: infer from events. |
| output_language | No | Language for analysis output. Default: English. |
| depth | No | `quick` (top-3 candidates, light validation) or `deep` (5+ candidates, full three-engine per candidate). Default: `deep`. |

**Minimum observation threshold**: this skill requires **>= 3 observed events** involving the same agent(s). Single-event abductive reasoning is unreliable. If the user provides only 1-2 events, warn explicitly and either (a) ask for more events, or (b) proceed with a heavy `strength: low` caveat on all outputs.

---

## Prerequisites

### Character Cards

This skill reads character cards from `characters/psychohistory/` as **data inputs** for the [PSY] engine. It does not generate cards.

- **If a card exists** for a relevant agent -> load it
- **If a card does not exist** -> inform the user: *"No character card found for [agent]. For deeper analysis, generate one first using `character-toolkit`. Proceeding with publicly known behavior patterns (shallower [PSY] analysis)."*
- **If a card exists but `data_cutoff` is stale** -> flag it and offer incremental Research Hand-off

### Card Freshness Check

For each loaded card, compare its `data_cutoff` against the earliest observed event:

```
IF card.data_cutoff < earliest_event_date - 6_months:
  -> Flag: "Card for [agent] last updated [date]. Events are from [date range].
     Recommend incremental research to update the agent's profile."
  -> Offer: Research Hand-off for delta (actions, statements, known shifts since cutoff)
  -> If user declines: proceed with existing card, tag [PSY] reasoning
     as `strength: low (stale card)`
```

---

## Execution Flow

### Phase 0: Define the Observation Set

Confirm with the user:

1. **What are the events?** List each event with date, actors, and what happened.
2. **What connects them?** Why does the user think these are related? (User intuition is a starting signal, not a constraint.)
3. **Focus question**: What does the user want to understand?
4. **Depth**: Quick or deep?

If >= 3 events with sufficient context -> proceed directly to Phase 1.
If < 3 events -> warn about unreliability, ask for more, or proceed with caveat.

---

### Phase 1: Fact Verification

**Before theorizing, verify the facts.** Read `../research-handoff/SKILL.md` and follow its protocol.

Research prompt template (fill and pass to research-handoff):

```
I need to verify and enrich the following set of events for abductive analysis.

For EACH event below:
1. Confirm it occurred — date, actors, key details
2. Note any important context the initial description missed
3. List related events in the same time window (+-2 weeks) that might be connected

Events to verify:
[list each event with user-provided description]

Additionally:
- Are there other significant events involving the same actors in this period?
- What are the main expert/media interpretations of this pattern (if any)?

Return findings organized by event, with sources cited.
```

After results return, update the observation set: correct factual errors, add missing events. Report the verified set to the user before proceeding.

---

### Phase 2: Agent Identification and Card Loading

#### 2.1 Identify Relevant Agents

From the verified observation set:
- **Primary agents**: who made the decisions that produced these events?
- **Secondary agents**: who influenced the primary agents or was the decision target?

For each, determine type: entity (personal/organization) or collective.

**Personal vs. Organization typing**: Does the agent's decision function include "personal political gain" as a variable independent of national/organizational interest? If yes → model as **personal entity** (captures the divergence between personal and institutional incentives). If no → model as **organization/leadership** (institutional logic only, no unnecessary modeling complexity). This determines which character-toolkit prompt to use (prompt-01 for personal, prompt-02 for organization).

#### 2.2 Load Character Cards

For each primary agent:
1. Check `characters/psychohistory/{agent_id}.json`
2. If found -> load and validate against `references/character-schema.md`
3. If not found -> inform user (see Prerequisites)
4. If found but stale -> run card freshness check (see Prerequisites)

#### 2.3 Map Hard Constraints

Briefly identify objective constraints bounding the agents' decision space: geographic, military, economic, institutional, temporal. These feed into the [GT] engine during validation.

---

### Phase 3: Candidate Theory Generation

The core abductive step. **A "theory" in this system is a specific perturbation on one or more agents' cognitive frameworks** — not a vague narrative.

#### 3.1 Generation Rules

1. **Generate >= 3 candidate theories**, ideally 4-6 for deep analysis
2. **Each theory must be expressible as a concrete framework perturbation**:
   - Which agent's model is being perturbed?
   - What specific `mental_models` / `decision_heuristics` are modified or replaced?
   - What changes in `core_objectives` or `priority_ranking` does this imply?
3. **Diversity requirement** — candidates must span different explanatory types:
   - At least 1 **strategic-rational** theory (the agent is optimizing something specific)
   - At least 1 **domestic/organizational** theory (driven by internal dynamics, not external strategy)
   - At least 1 **reactive/contextual** theory (response to a specific trigger, not proactive strategy)
   - **Null hypothesis must always be considered**: "these events are coincidental / separately motivated"
4. **Do not lock on the "most obvious" theory first** — this is the confirmation bias trap. Generate the full candidate set before evaluating any.

#### 3.2 Theory Expression Format

For each candidate:

```
### Candidate [N]: [Short title]

**Core claim**: [One sentence — what this theory asserts about the agent's decision model]

**Framework perturbation**:
- Agent: [agent_id]
- Modified element: [mental_models / decision_heuristics / core_objectives / priority_ranking]
- Standard framework says: [what the current card says]
- This theory says instead: [the specific modification]

**Predicted signature**: [What pattern would this theory produce? Does it match observations?]
```

#### 3.3 User Collaboration

Present the initial candidate set to the user. The user may:
- Add candidates the system missed
- Reject candidates with reasoning
- Merge similar candidates
- Refine framework perturbation descriptions

Incorporate user input, tag `source: user`.

---

### Phase 4: Theory Validation (Three-Engine Analysis)

For each candidate theory, **read `../three-engine/SKILL.md` and apply its [GT] / [PSY] / [ORG] framework**.

#### 4.1 Validation Procedure (per candidate)

1. **Apply the candidate's framework perturbation** to the relevant agent's profile
2. Run three-engine analysis: "Under this modified framework, would the agent have produced the observed events?"
3. Check each observed event:
   - Theory predicts this as natural/likely -> **consistent**
   - Theory predicts something contradictory -> **inconsistent**
   - Theory is silent on this event -> **neutral**
4. **Surplus predictions**: does the theory predict events that did NOT happen? Strong unfulfilled predictions weaken the candidate.
5. For agents with loaded character cards: [PSY] engine uses the **modified** mental_models from the candidate theory, not the standard card

#### 4.2 Cross-Event Consistency Score

Per candidate, tally:
- Events explained (consistent): +1
- Events contradicted (inconsistent): -2
- Events not addressed (neutral): 0
- Surplus predictions not observed: -1

This is an ordinal heuristic for ranking, not a precise score.

#### 4.3 Engine Convergence Check

Note where [GT], [PSY], and [ORG] agree or disagree on plausibility:
- **Three engines converge**: high confidence
- **Two agree, one disagrees**: note the dissent
- **No convergence**: genuine uncertainty — flag as "assessment uncertain"

---

### Phase 5: Ranking and Output

#### 5.1 Rank Candidates

Ranking criteria (in priority order):

1. **Cross-event consistency**: explains more observed events without contradicting any
2. **Engine convergence**: three-engine agreement ranks higher
3. **Historical precedent**: candidates backed by historical analogues rank higher
4. **Parsimony**: among equally explanatory candidates, simpler perturbation ranks higher
5. **Surplus prediction fit**: fewer unfulfilled surplus predictions ranks higher

#### 5.2 Identify Discriminating Events

For the top 2-3 candidates, identify **future observable events that would distinguish between them**:

```
If [Theory A] is correct -> expect [X] within [timeframe]
If [Theory B] is correct -> expect [Y] within [timeframe]
If [X] happens but not [Y] -> Theory A more likely
If [Y] happens but not [X] -> Theory B more likely
```

These discriminators are the most actionable output.

#### 5.3 Output Format

```
# Observation Set (Verified)
- [Event 1]: [date] — [description]
- [Event 2]: [date] — [description]
- [Event 3]: [date] — [description]
...

## Agents Analyzed
- [Agent 1] (card: [loaded/not found/stale]) — [role in events]
- [Agent 2] ...

## Ranked Candidate Theories

### Rank 1: [Theory title]
**Core claim**: ...
**Framework perturbation**: ...
**Consistency**: [N/M events explained, 0 contradicted]
**Engine assessment**:
- [GT]: ...
- [PSY]: ...
- [ORG]: ...
**Historical precedent**: [if applicable]

### Rank 2: [Theory title]
...

### Rank 3: [Theory title]
...

[### Null hypothesis: Events are separately motivated]
...

## Discriminating Events (What to Watch)
- If Rank 1 is correct -> expect [X] by [date]
- If Rank 2 is correct -> expect [Y] by [date]
- Decisive signal: [what event would definitively confirm/deny the top candidate]

## Epistemic Boundaries
[Required honesty declarations — see below]
```

---

### Phase 6: Interaction (Ongoing)

#### 6.1 Fact Injection
User: "Breaking news — XXX just happened"
-> This event **has occurred**. Add to the observation set, re-run Phase 4 for each candidate against the new event, update rankings.

#### 6.2 Add Candidate
User: "You missed a possibility — XXX"
-> Accept the new candidate, express as framework perturbation, run Phase 4 validation, insert into ranking.

#### 6.3 Refine Candidate
User: "Theory 2 should also include X"
-> Modify the framework perturbation, re-run Phase 4, update ranking.

#### 6.4 Pivot to Forward Analysis
User: "OK, assuming Rank 1 is correct — what happens next?"
-> Hand off to the main Psychohistory skill with Rank 1's framework perturbation loaded as the agent's working model.

#### 6.5 Deep Test a Candidate
User: "Test Theory 2 more rigorously"
-> Hand off to `theory-test` skill with the candidate's framework perturbation as the theory input, gathering additional test events via Research Hand-off.

---

## Epistemological Boundaries (Required in Every Output)

1. **Underdetermination**: "Multiple theories can explain the same observations. The top-ranked candidate is the *best current explanation*, not a proven cause."

2. **Observation set limitation**: "This analysis is based on [N] observed events. Additional observations could change the ranking."

3. **Open model space**: "The true decision-making model may not be among the candidates listed. These are the most plausible explanations *given what we considered*."

4. **Card dependency caveat** (when applicable): "Agents [X, Y] were analyzed without character cards. [PSY] analysis is shallower — conclusions tagged `strength: low`."

5. **Stale card caveat** (when applicable): "Agent [X]'s card predates the events by [N months]. [PSY] reasoning may miss recent shifts."

---

## Quality Standards

| Good Analysis | Bad Analysis |
|---|---|
| >= 3 diverse candidates with explicit framework perturbations | 1 "obvious" theory presented as the answer |
| Each candidate validated against ALL observed events | Cherry-picks events fitting the preferred theory |
| Null hypothesis explicitly considered | Assumes events must be connected |
| Discriminating future events identified | No "what to watch" output |
| Epistemic boundaries prominently displayed | Top candidate presented as established fact |
| [GT] / [PSY] / [ORG] tags on each reasoning line | Undifferentiated narrative |
| Historical precedents cited where available | Pure speculation |

---

## Never Do

- **Never present abductive inference as causal proof** — "best explanation" != "true explanation"
- **Never run single-event inference without explicit caveat** — >= 3 events minimum for reliability
- **Never lock on the first plausible theory** — generate the full candidate set before evaluating
- **Never skip the null hypothesis** — "coincidence / separate motivations" must always be a candidate
- **Never use unvalidated character cards** — load from `characters/psychohistory/` or proceed without
- **Never fabricate historical precedents** — if no close match exists, say so
- **Never output precise probability numbers** — rank only, no quantification

---

## Worked Example Sketch

*Illustrates the flow, not a complete analysis.*

**Observation set** (hypothetical):
1. Vietnam PM visits Beijing — signs infrastructure cooperation deals (Week 1)
2. UAE President visits Beijing — energy partnership expansion announced (Week 1)
3. Spain PM visits Beijing — trade talks amid EU-China tensions (Week 1)
4. Russia FM visits Beijing — "strategic coordination" communique (Week 2)

**Focus question**: What decision-making model explains China hosting four high-level visits in rapid succession?

**Agents**: Xi Jinping / Chinese leadership (primary), each visiting leader (secondary)

**Candidate theories** (sketch):

1. **Anti-US economic coalition building** — China is assembling a diversified partner network as a direct trade war response. Perturbation: Xi's `priority_ranking` shifts from "stable bilateral management" to "active coalition formation."

2. **Global mediator positioning** — China is projecting itself as the responsible great power, offering an alternative to perceived US unilateralism. Perturbation: Xi's `mental_models` adds "global governance vacuum = opportunity."

3. **Diplomatic tempo as signaling** — The clustering is intentional: showing audiences that China is "not isolated." Perturbation: none deep — this is a standard [GT] signaling move under existing framework.

4. **Null hypothesis** — Each visit has its own bilateral logic; the clustering is calendar coincidence.

**Validation** would check each theory against all four visits: why *these* four countries? Why *this* timing? Which theory best explains the selection and sequencing?

**Discriminators**: If Theory 1, expect anti-US coordination (joint sanctions statements, alternative payment systems). If Theory 2, expect China mediating conflicts where it previously stayed neutral. If Theory 3, expect the tempo to slow once the signaling window closes.
