# 新闻解读器（News Interpreter）

[English](./README.md) · **中文**

> 溯因推理引擎：给定一组已观测事件，生成并排序候选理论来解释背后的模式。每个理论是对 agent 认知框架的结构化扰动——可验证，不是叙事。

## 做什么

输入一组相关事件（>= 3 个），从观测反推决策模型。生成多样化的候选理论，用三引擎分析 [GT] / [PSY] / [ORG] 逐个验证，按解释力排序，并识别能区分顶级候选的未来事件。

## 使用场景

- **外交集群**："四国同周访华——背后什么战略逻辑？"
- **意外政策转向**："巴基斯坦军方入驻沙特空军基地——这揭示了什么？"
- **矛盾模式**："这几个贸易决定看起来矛盾——什么框架能让它们一致？"
- **事件解读**："刚发生了这件事——合理的解释有哪些？"

## 安装

本 skill 在 Psychohistory monorepo 的 `skill/news-interpreter/` 目录下。独立使用时复制本目录，并确保 `skill/three-engine/` 和 `skill/research-handoff/` 也可用（通过 Method A 引用）。

## 依赖

- `three-engine` — 用于对每个候选理论做 [GT] / [PSY] / [ORG] 验证
- `research-handoff` — 用于事实核实和增量卡片更新
- `characters/psychohistory/` 中的角色卡 — 数据输入（非 skill 依赖）

## 被谁使用

- 心理史学主场景 skill（模式 2 路由）
- 任何需要从观测反推模型的工作流
