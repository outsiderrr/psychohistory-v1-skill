# 理论检验（Theory Test）

[English](./README.md) · **中文**

> 假设验证引擎：给定一个关于 agent 决策模型的具体理论，用双轨三引擎分析对历史事件进行检验。对比标准框架和替代框架，识别能区分二者的未来事件。

## 做什么

接收用户提出的假设（表达为认知框架扰动），与 agent 的标准角色卡做结构化对比。用 [GT] / [PSY] / [ORG] 分析同一批历史事件，产出逐事件对比矩阵和区分性未来事件。

## 使用场景

- **隐藏动机**："我觉得美联储的真实目标是保护资产价格，不是控制通胀"
- **幕后影响**："我觉得 Trump 被 Thiel 通过 Vance 影响的程度超过表面"
- **内部动态**："我觉得伊朗温和派实际上比公开姿态更强"
- **战略意图**："我觉得这次重组实际上是在为收购做准备"

## 安装

本 skill 在 Psychohistory monorepo 的 `skill/theory-test/` 目录下。独立使用时复制本目录，并确保 `skill/three-engine/` 和 `skill/research-handoff/` 也可用（通过 Method A 引用）。

## 依赖

- `three-engine` — 用于双轨 [GT] / [PSY] / [ORG] 分析
- `research-handoff` — 用于收集历史测试事件
- `characters/psychohistory/` 中的角色卡 — 作为标准框架基线（必需）

## 被谁使用

- 心理史学主场景 skill（模式 3 路由）
- `news-interpreter` — 内部对每个候选理论执行同样的验证逻辑
- 任何需要对 agent 模型做严格假设检验的工作流
