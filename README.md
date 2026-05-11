# prompts

这里存放我自己持续打磨的一些 prompt，主要是面向代码审计、设计规约推导，以及若干辅助性的工作流 prompt。

这些文件本质上都是可复用的 prompt 模板。日常会优先维护当前版本，旧版本保留作对照，部分主题会单独维护中文场景化版本。

## 目录说明

- `current/`: 当前维护中的版本，默认应优先使用这里的内容。
- `old/`: 旧版本归档，用来回看措辞、结构或方法演进。
- `sp-ver/`: 面向特定场景整理的中文版本，目前主要是 Hoare 系列的中文化 prompt。

## 主要内容

### Hoare 系列

- `hoare-prompt`: HoarePrompt 方法本身的定义或主提示。
- `hoare-design`: 当缺少设计文档时，从实现和调用方式反推描述性规约。
- `hoare-audit`: 在已有规约前提下做持续正确性审计，把非决策问题和决策性问题分开处理。

这三个 prompt 可以连起来用：先定规约，再做审计，再迭代修正。

### 工作流系列

- `workflow`: 通用开发工作流程规范——三阶段（调研 → 架构 → 细化）+ 实现 + 修复迭代 + 文档管理。面向新功能开发与 Bug 修复的标准流程，AI 助手的行为准则与禁令。
- `principle-workflow`: `workflow` 的演进版——在三阶段之上叠加一层**§0 精神宪法**。糅合 `principle-derivation` 的"问题意识 → 派生原则 → 推导"骨架，把 `principles.md` 作为不可篡改的宪法，所有下游设计 / 代码 / 报告必须 100% 满足。
- `workflow-audit`: 多方向、disprove-first 的 PR 审计工作流，并行 challenge 轮 + evidence-gated 结论。

#### `workflow` 与 `principle-workflow` 的演进关系

| 维度 | `workflow` | `principle-workflow` |
|---|---|---|
| 起点 | 调研阶段 | **§0 精神宪法**（先于调研） |
| 顶层文档 | `architecture.md` | `principles.md`（宪法）→ `architecture.md`（受宪法约束） |
| 决策溯源 | 每条决策有理由 | 每条决策必须**追溯到宪法的某条原则**，写入推导表 |
| 设计修订 | 修订设计文档 | 区分修订层级（细化/架构/宪法）；宪法修订需用户显式拍板 |
| AI 助手禁令 | 不擅自改测试用例、不跳过设计 | 额外：**禁止修改 `principles.md` 未经允许** |
| 适用场景 | 一般功能开发 | 项目目标存在结构性张力、需要稳定承诺的根本意识 |

**何时用哪个**：

- 简单功能、CRUD、局部模块 → `workflow` 够用
- 项目立项 / 需要长期稳定承诺 / 多人协作易漂移 / 已多次出现"为某次变更临时改架构"现象 → 用 `principle-workflow`，先把宪法立住
- 已有 `workflow` 项目升级 → 反向 reverse-engineer 出 `principles.md`，迁移到 `principle-workflow`（详见 `principle-workflow.md` §0.4）

`principle-workflow` 的核心增量：**以宪法换稳定性**——多一层约束，但下游的所有争议都能溯源到宪法层判定，避免讨论无限循环。

### 辅助 prompt

- `finegrained-check`: 适合做更细粒度的检查或补充验证。
- `evo-graph`: 用来梳理演进关系、推导路径或结构变化。
- `make-survey-plan`: 用来设计 survey / organise / plan 类型的调研与整理流程。

## 使用约定

- 如果只是直接使用 prompt，优先从 `current/` 读取。
- 如果要比较 prompt 的演变过程，再去看 `old/`。
- 如果目标场景本身就是中文语境，优先看 `sp-ver/` 里的中文版本。

## 维护原则

- 新增或修改时，优先收敛到 `current/`。
- `old/` 不追求同步，只保留有参考价值的历史版本。
- 中文版本只在确实需要本地化表达、术语映射或场景适配时单独维护。