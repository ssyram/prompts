# QPDI Series — 设计架构

## Task Overview

本设计承接 [`principles.md`](principles.md) 的需求，说明 `/qpdi` 与 `/qpdi-compose` 如何共同提供最小充分的认知和写作能力。运行期实现是 [`current/qpdi.md`](../../current/qpdi.md) 与 [`current/qpdi-compose.md`](../../current/qpdi-compose.md)；draft 是候选实现，不在本文中伪装成现行版本。

## 1. 核心模型

```text
Q = Q.I + Q.A + Q.E

root: Q.I + relevant Q.A/Q.E
child: parent contract + applicable global P + relevant Q.A/Q.E
          └─ R.Derivation ─→ P

D-frame = P + M + R
M = components + composition
          └─ R.Satisfaction ─→ M ⊨ P

I realizes/refines leaf M and preserves leaf P
```

核心内容角色只有 `Q.I / Q.A / Q.E / P / M / R / I`。root、component、leaf、local/global、candidate/confirmed 是位置、作用域或状态；pre/post、invariant、禁止条件是 P 的表达形式。

### Q

Q 是问题意识面向当前对象形成的有限、可论证结构。它的对象指向可由 intentionality 理解，但 Q.I 的规范意图不是 intentionality 本身；Q.I、Q.A、Q.E 分别承担判断方向、认知前提和经验依据，其区分来自论证功能而非人格类型。

- **Q.I** 定义 correctness、wanted outcome 与 boundary；三者是观察面，不是三种新本体。
- **Q.A** 是被采纳并参与 R 的认知命题。
- **Q.E** 是观察、样本和经验材料。
- 经验被归纳成认知命题时使用 `Q.A supported-by Q.E`，不在两处复制同一句话。

Q 的 whole view 是阅读组织：材料确有整体关系时呈现 Q.I 的展开、限制或张力，并把 Q.A/Q.E 放回实际语境。它通常位于 Overview；只有用户确认其本身定义规范方向时才进入 Q.I。

### P

P 是当前 frame 拥有的全部可判定设计性质：

- `scope = owner`：只约束 owner frame，常称 local P；
- `scope = descendants(selector)`：对命中后代生成 obligation，常称 global P。

父层从 P 投影 component contract，不把整组 local P 复制给 child。普通未规定内容只是 P 的补空间；必须保护的自由度本身写成 P。

### M

M 由 component occurrences 和 composition 构成。Component 记录本层需要的 role 与 allocated contract；composition 说明控制、数据、状态、顺序、失败和责任如何形成整体行为。只有组件清单而没有 composition，不能构成完整设计模型。

同一对象在父 M 中是 component，展开后成为自己的 D-frame。父层引用 child，不内联完整 child P/M/R。

### R

R 包含：

- **Derivation**：root 从 Q 得到 P；child 从 parent contract 与适用 global P 得到 P；Q.A/Q.E 只提供认知和经验支撑。
- **Satisfaction**：component contracts 与 composition 作为 whole 证明 `M ⊨ P`，覆盖接口、共享前提和已触发 global P。
- **Optimality**：存在真实候选时解释为何当前 P/M 更合适；没有候选时不制造比较。

R 提供对象级论证；SCCO 检查来源、覆盖、重复和候选比较是否充分。R 需要新约束时必须写回 P，不能把它藏在证明文字中。

### I

I 是具体代码、配置、字段、命令、部署物或文案。它具体化叶子 M，并必须保持叶子 P；R 不被实现，而是说明为什么这些关系应当成立。测试、命令和实际产物提供符合性证据。

两个选择不同但所有 P 与 M 的可观察行为不变时，差异属于 I；若改变行为、跨组件 contract 或风险边界，仍属于 P/M。

## 2. 设计性质

### P.1 规范来源不旁路

只有 D.root 直接从 Q.I 取得规范方向。Child 的直接 obligation 来自 parent contract 与适用 global P；它可以使用相关 Q.A/Q.E，但不能绕过父层重解 Q.I。

### P.2 规则具有真实 owner 和 scope

Sibling 的 owner-scope P 不横向泄漏。共同设计约束提升到公共祖先 P，共同领域认知进入 Q.A；传播必须有明确 selector，不能凭“位置高”推断。

### P.3 局部成立不推出 whole 成立

Component 分别满足 contract，只是 Satisfaction 的前提。Frame 仍须检查 contracts 是否覆盖全部义务、composition 是否处理接口和共享前提、global P 是否 discharge，以及整体实际保证是否成立。

### P.4 Guarantee narrowing 逐层向上

实际 guarantee 是 R.Satisfaction 对当前 M 可证明性质的结果，不是新内容类型。局部变化后：

1. owner 比较实际 guarantee 与 parent contract；
2. 仍满足则只是本层 redesign，传播停止；
3. 变弱则只把差额向 parent 报告，由 parent 重算 composition；
4. D.root 才把最终差额与 Q 比较，并由人决定是否修改 Q；
5. narrowing 不自动向下削弱既有 child contract。

设计限制、残余风险和 trade-off 留在拥有该方案的 D-frame。人允许带着可见限制继续，不等于人改变了 Q.I。

### P.5 展开后必须 re-unify

分析可以展开候选、反例和中间概念；交付前要合并同义项，把表达形式收回父概念，把位置、角色、状态和证据移出本体，并删除无消费者内容。Re-unification 不能删除决定结论的前提、边界或分歧。

### P.6 使用形式服从用户

静态模型保持稳定；可见 header、字段、目录和步骤只是推荐。简单问题不展示完整脚手架，用户指定的范围、格式和授权优先。

## 3. 两份 prompt 的分工

### `/qpdi`

提供静态模型和轻量应用能力：识别论域与话语动作，保全 proof，锚定 Q/P/M/R/I，检查 Structural Locality、SCCO 与上下游影响，最后 re-unify。它可以指导分析和执行，但不自动产生固定工作流。

### `/qpdi-compose`

把材料写成 Task Overview、条目化 Q 与递归 D-frame。它自包含写作所需的最小核心，因为可能被单独调用；但核心定义保持简短，不复制 `/qpdi` 的完整讲解。

Compose 不生成 I，也不运行完整 SCCO tribunal。它负责把叶子 D 写到足以交接 I，并在授权后写入 Q/P/D 文件。

### 相关 prompt

- `/skill-routing`：决定何时选用 `/qpdi`、Compose 或专项审查；不重新定义 QPDI。
- `/scco-recall`、`/qpdi-tribunal`、`/hoare-audit`：承担系统性检查或裁决；不把其流程复制到 Compose。

## 4. Compose 的最小写作模型

```text
Task Overview
→ Q.I / Q.A / Q.E
→ D.root { P, M, R }
→ child D-frame { P, M, R } ...
→ leaf: implemented-by I
```

每个 component 在定义处明确：

- `expands-to`：仍需独立说明语义、分支、失败、composition 或跨工件不变量；
- `implemented-by`：这些问题已在当前 frame 解决，指向或声明承接 contract 的 I。

树可以不平衡。叶子 D 的停止条件不是“文件已经够长”，而是现有 P/M/R 已足以判断 I 是否正确承接。

## 5. 文档 ownership

落盘顺序是：用户指定位置 → 既有稳定文档结构 → 能完整拥有该功能的最小稳定 `<OWNER_DIR>`。专项任务位于 owner 下的专项位置；不因仓库根存在 `docs/` 就默认污染整体文档根。

文件名和层数只是视图。优先复用既有 `docs/`、`design/`、`spec/`、ADR 或 feature 文档；多个 owner 代表实质不同选择且无法从项目判断时才询问用户。

## 6. 状态与授权

三条状态轴彼此独立：

- 内容采纳：candidate / confirmed / superseded；
- 事实证据：verified / pending / contested；
- 文件操作：response-only / write-authorized / written。

确认内容不等于事实已验证或允许写文件；写入授权也不确认 candidate Q。用户质询促成 AI 自我修正，不自动构成用户的新设计裁决。

## 7. D → prompt obligations

| 设计义务 | `/qpdi` | Compose |
|---|---|---|
| Q.I/Q.A/Q.E 分工与 whole view | 完整认知 | 可直接写的最小格式 |
| `D-frame = P + M + R` | 静态定义 | 递归模板 |
| Structural Locality 与 P.scope | 来源和传播规则 | owner/scope/contract 字段 |
| whole-level Satisfaction | SCCO 与设计认知 | frame 完成检查 |
| guarantee narrowing | 上下游归层规则 | 暴露差额，不静默改 Q |
| Discover 与 re-unification | 完整规则 | 写前归层、写后压缩 |
| I 边界 | 认知与验证关系 | 停止条件和 handoff，不生成 I |
| 文件 ownership | 仅说明不是本体 | 实际路径选择与写入授权 |

## 8. 验证边界

文本结构只能使错误可检查，不能保证模型、事实或读者必然正确。替换 official 前至少应覆盖：多 Q.I 且不可强行统一、局部 redesign 被本层吸收、真实 narrowing 逐层上报、局部 pass 但 whole fail、以及 component 在展开和 I handoff 之间正确停止。
