# News Interpreter

**English** · [中文](./README_CN.md)

> Abductive inference engine: given a set of observed events, generate and rank candidate theories explaining the underlying pattern. Each theory is a structured perturbation on agents' cognitive frameworks — testable, not narrative.

## What it does

Takes a cluster of related events (>= 3) and works backward from observations to decision-making models. Generates diverse candidate theories, validates each using three-engine analysis [GT] / [PSY] / [ORG], ranks by explanatory power, and identifies future events that would discriminate between top candidates.

## Use cases

- **Diplomatic clusters**: "Four countries visited China this week — what strategic logic connects them?"
- **Unexpected policy shifts**: "Pakistan military deployed to Saudi air base — what does this reveal?"
- **Contradictory patterns**: "These trade decisions look contradictory — what framework makes them consistent?"
- **Event interpretation**: "This just happened — what are the plausible explanations?"

## Install

This skill lives in the Psychohistory monorepo at `skill/news-interpreter/`. To use standalone, copy this directory and ensure `skill/three-engine/` and `skill/research-handoff/` are also available (referenced via Method A).

## Dependencies

- `three-engine` — for [GT] / [PSY] / [ORG] validation of each candidate theory
- `research-handoff` — for fact verification and incremental card updates
- Character cards in `characters/psychohistory/` — data input (not a skill dependency)

## Used by

- Main Psychohistory scenario skill (Mode 2 routing)
- Any workflow that needs to reason backward from observations to models
