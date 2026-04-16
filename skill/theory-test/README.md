# Theory Test

**English** · [中文](./README_CN.md)

> Hypothesis validation engine: given a specific theory about an agent's decision-making model, test it against historical events using dual-run three-engine analysis. Compare the standard framework against the alternative, and identify future events that would discriminate between them.

## What it does

Takes a user-provided hypothesis (expressed as a cognitive framework perturbation) and runs a structured comparison against the agent's standard character card. Tests both frameworks against the same historical events using [GT] / [PSY] / [ORG] analysis, produces a side-by-side comparison matrix, and outputs discriminating future events.

## Use cases

- **Hidden motives**: "I think the Fed's real goal is protecting asset prices, not controlling inflation"
- **Shadow influence**: "I think Trump is being influenced by Thiel through Vance"
- **Internal dynamics**: "I think Iran's moderate faction is stronger than the public posture suggests"
- **Strategic intent**: "I think this restructuring is actually prep for an acquisition"

## Install

This skill lives in the Psychohistory monorepo at `skill/theory-test/`. To use standalone, copy this directory and ensure `skill/three-engine/` and `skill/research-handoff/` are also available (referenced via Method A).

## Dependencies

- `three-engine` — for dual-run [GT] / [PSY] / [ORG] analysis
- `research-handoff` — for gathering historical test events
- Character cards in `characters/psychohistory/` — required as the standard framework baseline

## Used by

- Main Psychohistory scenario skill (Mode 3 routing)
- `news-interpreter` — internally runs the same validation logic per candidate theory
- Any workflow that needs rigorous hypothesis testing against agent models
