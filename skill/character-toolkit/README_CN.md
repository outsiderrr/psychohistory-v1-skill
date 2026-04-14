# 心理史学 · 角色卡生成工具包（总索引）

[English](./README.md) · **中文**

> 本工具包包含生成心理史学所有类型角色卡的标准提示词。
> 根据你要生成的角色类型，选择对应的提示词文件。

---

## 角色类型路由

```
你要生成什么？
│
├── 一个具体的人物？（特朗普、内塔尼亚胡、鲍威尔）
│   └── 👉 使用 prompt-01-personal-entity.md（调用女娲skill）
│
├── 一个组织/机构？（美联储、革命卫队、苹果公司、伊朗政府）
│   └── 👉 使用 prompt-02-org-entity.md
│
├── 一个群体？（MAGA选民、美股投资者、伊朗民众）
│   └── 👉 使用 prompt-03-collective.md
│
└── 角色之间的关系？（特朗普与万斯、伊朗政府与革命卫队）
    └── 👉 使用 prompt-04-relationship.md
```

---

## 文件清单

| 文件 | 用途 | 输出格式 |
|------|------|---------|
| `prompt-01-personal-entity.md` | 个人实体角色卡（调用女娲skill） | `[agent_id].json` |
| `prompt-02-org-entity.md` | 组织实体角色卡（政府/公司/军事组织） | `[agent_id].json` |
| `prompt-03-collective.md` | 群体角色画像 | `[agent_id].json` |
| `prompt-04-relationship.md` | 角色间关系定义 | `[rel_id].json` |

---

## CLI-first 设计

本工具包**默认在 CLI 环境**（例如 Claude Code）下运行。所有 phase-based prompt 和 `SKILL.md` 都假设以下能力可用：

- 文件系统读写
- `git rev-parse --show-toplevel` 用于自动定位项目根目录
- `python3` + `jsonschema` 用于自动 schema 校验
- `WebFetch` 用于获取研究阶段的源数据

**对话 AI 作为逃逸通道**：如果你想在 ChatGPT / Claude.ai / Trae / Gemini 等对话 AI 里生成一张卡，**不要直接复制 `prompt-0X` 文件**——那些是为 CLI 写的。而是通过 `SKILL.md` 的 **Export 模式**：在 CLI 里调用它，告诉它目标和场景，它会输出一段适配过的自包含 prompt 供你复制到对话 AI 里执行。对话 AI 的响应（references.md + JSON）带回 CLI 后，本工具包会做校验和落盘。

这个设计让 CLI 下的每一步都可以放心使用文件系统、Python 校验、WebFetch 等能力，不必为"万一在对话 AI 里跑"做让步。对话 AI 的支持通过 Export 模式单点解决。

---

## 通用规则

以下规则适用于所有类型的角色卡：

1. **所有JSON必须通过标准解析器验证** — 不允许有语法错误
2. **所有文本内容使用英文** — name、description等字段一律英文
3. **日期格式一律 YYYY-MM-DD** — 不接受不完整日期
4. **agent_id 一律小写字母+连字符** — 如 `us-federal-reserve`、`maga-base`
5. **每张卡必须有 honesty_boundaries** — 明确说明这张卡做不到什么

---

## 方法论原则（v1.1）

以下 5 条原则贯穿所有 **phase-based prompts**（prompt-02 / 03 / 04），是整个角色卡生成方法论的基础。prompt-01（个人实体）因为依赖 Nuwa 的多阶段蒸馏管线，继承了其中的原则 1、3、4、5；原则 2 同样适用。

1. **两阶段最少** — 研究阶段先行、结构化阶段在后。绝不允许从空白直接跳 JSON
2. **references.md 是主产物** — JSON 是它的压缩索引。先写笔记 → 按它编译 JSON → 验证每个 JSON 字段都能指回 references.md 中的具体证据
3. **证据强度分级** — 每条结论标注 `strength: high / medium / low`。`high` 要有多案例支撑，`low` 可用但必须显式标明
4. **显式认知边界** — `honesty_boundaries` 必须包含类型专属声明：
   - 组织：派系分化声明
   - 群体：内部异质性声明
   - 关系：场景绑定声明
5. **历史先例优先** — 有历史案例支撑的推理排在纯理论推理前面；无先例的推论标 `strength: low`

这 5 条是**硬性要求**，不是建议。任何 phase-based prompt 执行后产出的 JSON 如果违反其中任何一条，必须回到对应 Phase 补充证据或修正分级。

---

## 保存路径

| 类型 | 路径 |
|------|------|
| 女娲原始数据 | `characters/nuwa/[agent_id].md` |
| 个人实体JSON | `characters/psychohistory/[agent_id].json` |
| 组织实体JSON | `characters/psychohistory/[agent_id].json` |
| 群体角色JSON | `characters/psychohistory/[agent_id].json` |
| 关系定义JSON | `characters/relationships/[rel_id].json` |
| 索引文件（论证过程） | 与对应JSON同目录，后缀为 `.references.md` |

每张角色卡和每份关系定义都必须同时生成一份索引文件（`.references.md`），记录每个结论的证据链条。索引文件是给需要验证或修改结论的用户看的，不影响引擎运行。

---

## 快速判断：这个角色是实体还是群体？

问自己一个问题：**这个角色有没有一个"最终拍板人"？**

- 有 → 实体（entity）。即使组织很大，只要有一个人或一个小班子能拍板，就是实体。
- 没有 → 群体（collective）。没有人能代表整体做出决定，行为是统计性的涌现结果。

**灰色地带怎么办？**

有些角色介于两者之间，比如"伊朗最高安全委员会"——它有集体决策机制但没有单一独裁者。这种情况下建议按**组织实体**建模，但在卡片中特别标注内部决策机制的特殊性。
