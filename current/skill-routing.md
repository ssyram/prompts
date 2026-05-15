# skill-routing — 用哪个 skill 的经验记录

> 不是 skill 规约，是**使用 skill 时的选择经验沉淀**。每次发现选错了 skill 就回这里加一条。
> 这个文件本身不被 `/<name>` 调用——它是给主对话里的 Claude 看的 routing 参考。

---

## principle-derivation v1 vs v2

两个 skill 同骨架（意识 → 原则 → 推导），但姿态相反。

| 场景 | 选 |
|---|---|
| 已收敛分析的**事后整理** | **v1** |
| audit 报告、code review、设计回顾、bug 复盘 | **v1** |
| 引导读者**从零产生独立洞察** | v2 |
| 教学材料、概念入门、新人 onboarding | v2 |
| 自己设计要立结构 / 提原则 | v1 |
| 单层、扁平、原则少 | v1 |
| 多层认识在不同时刻形成，要展示这一过程 | v2 |
| 主体不清楚 / 还在共同探索 | 都不用，先 trial-and-error |

### v1 vs v2 选用案例：audit 报告

**情境**：审计 ciim2022p6 trajectory，3 份独立 reviewer 产出的 audit 报告被要求"用 principle-derivation-v2 语言重写"。

**结果**：审视者**已经知道答案** → v2 的"邀请同行"姿态变成"把事后回顾包装成意识展开"——principle-derivation-v2 §四警告的"用方法替代思考"几乎踩中。具体内容（commit hash / 字段值 / 覆盖检查结论）被叙述节奏遮蔽，决策者读起来反而不知道审视者具体发现了什么。

**判据**：审计 / 复盘这种"已收敛分析的事后陈述"用 v1（直接给结构 + 推导表）。v2 用于"还在展开中" / "邀请同行"。

**记忆点**：v2.md §六已经有这张选用表，**容易在被任务 prompt 推进时忘记查表**。下次启动任何"重写 / 重表达 / 重新组织已有内容"的任务，先回 v2.md §六对一下姿态匹配。

---

## workflow / workflow-audit / hoare-audit

三个 skill 名字都带 workflow / audit，容易混。

### 一句话区别

- **workflow** — 开发**做事**的规范，不审计任何东西
- **workflow-audit** — 审视一份 **GitHub PR** 的多方向并行 review（驳回优先）
- **hoare-audit** — 审视**代码 vs 设计 spec** 的正确性（spec 必需）

### 对比

| skill | 输入 | 输出 | 是否需要 spec | 主体关系 |
|---|---|---|---|---|
| workflow | 任务描述 | 设计 + 代码 + 测试 | 否（自己产生设计文档） | 做事的人 |
| workflow-audit | GitHub PR 编号 | review 报告 | 项目 workflow.md 优先，可降级 | 外部审查（不同主体）|
| hoare-audit | 代码 + 设计 spec | findings + auto-fix | **必需**（"No audit without a spec"） | 自审 / 他审都可 |

### 选用决策

- 在写新功能 → **workflow**
- 别人提了 PR 要 review → **workflow-audit**
- 拿到一份代码，想审"是否满足设计意图" → **hoare-audit**（先确认有 spec；没有 spec → 先建 spec 或拒绝 audit）

### 联系

- workflow 立 spec → hoare-audit 验代码满足 spec
- workflow-audit 跟"做事的人"是不同主体（外部 PR review）
- hoare-audit 跟"做事的人"可以是同一主体（自审）也可以是外审

### 容易混淆点

- **workflow 含的"代码审核"段（§4.2）**不是独立 audit skill——那是开发过程内的自审，完全在 workflow 内做完，不调 audit skill
- **workflow-audit vs ultrareview**：都是 PR review，但 workflow-audit 是单 agent 做 multi-direction；ultrareview 是 multi-agent cloud parallel（user-triggered 计费）。ultrareview 不能由 Claude 自己调起
- **hoare-audit vs trajectory-audit**（如 auto-proof-trajectory-audit）：前者审代码正确性 vs spec，后者审 agent 跑动轨迹是否符合设计精神。不同对象、不同输入

---

## audit 类 skill 的共性："驳回优先 / 先反后反反"

workflow-audit / hoare-audit / charter-craft §4.8 都共享一个机制——**审视者应先挑刺，再 counter-challenge**。

理论根据（charter-craft §4.8.1）：LLM 转换立场就能挑出问题（高 recall），单纯 confirm 几乎不发现问题（低 recall）；counter-challenge 过滤无效挑刺（高 precision）。反两次是经验最优点。

实际项目里：
- audit-principles.md 制宪用了 §4.8 先反后反反
- workflow-audit 直接把多方向并行 challenge 嵌入流程
- hoare-audit 用 spec 作为 ground truth 让 challenge 有可证伪 anchor

**判别**：写任何新 audit 流程，"先反后反反"应该是默认结构，不是 nice-to-have。

---

## 记入此文件的触发条件

- 用错了 skill / 用错了 skill 变体（如 v1/v2）发现的经验
- 两个看似可换的 skill 实际有清晰分工的情况
- skill 名字相似但定位不同的情况
- 通用项目工作流里反复证实的"在 X 场景该用 Y skill"判别
