# PR Craft — Pull Request 描述书写规范

> 让 PR 描述强迫作者把跨 PR 的设计哲学、不变式契约、风险边界都摆上台面，让 reviewer 不用读完代码也能判断该不该 merge。

---

## 适用范围

- **架构 / feature / 跨模块改动 / 设计哲学性 PR** → 全套 7 段
- **小修小补 / 1-3 文件改动 / 无设计决策** → 退化到 3 段（为啥 / 干了啥 / 测试）

> 写 typo fix PR 也套全 7 段是 overkill；按 PR 性质裁剪。

---

## PR 描述结构（7 段）

### 1. 为啥（Why）

当前痛点 — 用户 / 开发者 / 系统能感受到的具体问题，不是 "希望系统更好"。

**反模式**：
- "为了改进观测性"（vague）
- "重构 logging 模块"（描述方法不是问题）

**正确**：
- "tool gate 决策、文件读取、软门禁结果无法统一 grep；事件分散在 stderr 和 in-memory，跨进程重启就丢"（**具体 + 可被 reviewer 验证**）

链接相关 issue / 讨论 / 历史 commit。

### 2. 干了啥（What changed）

列改动而不是叙事。按子系统 / 文件分组，每条一句话。

**反模式**：
- 大段 prose 描述实现细节（reviewer 直接读 diff 更快）
- "重构了 X 模块"（不具体）

**正确**：
- 分小节（域 / 接口 / 数据流 / ...）
- 每个改动一句话 + 关键文件路径
- **量化**：行数 / 域数 / 测试数 / API 数

### 3. 干的原则（Design principles）

跨 PR 的设计哲学和不变式契约。这一段是稀缺的，绝大多数 PR 描述跳过。

**为什么必须写**：
- 强迫作者**主动**说清"为什么这种做法是正确的"，而不是事后被 reviewer 质问
- 6 个月后另一个工程师做类似改动时，可以直接看到当年的设计精神
- 让 reviewer 能判断"这个 PR 是否符合项目长期方向"，而不只是"代码语法对不对"

**包含子点**：
- **架构精神 / 设计哲学**（如"状态权威 vs 执行环境"）
- **不变式 / 契约**（必须保证什么不变，I1-IN）
- **奥卡姆剃刀声明（推荐）**：本方案是不违反原则情况下做到尽量精简的——主动声明"没过度设计"，避免 reviewer 怀疑"为什么不是 X 更复杂方案"
- **接受的代价（明示）**：哪些场景下 by design 容忍了 ugly behavior

### 4. 测试如何进行（Testing）

显式独立段，让 reviewer 不用猜怎么验证的。

**包含**：
- **范围**：unit / integration / e2e / fmt / clippy 各跑了多少
- **关键测试**：核心契约对应的测试名 + 测试覆盖什么场景（不是描述实现）
- **审计 trail（如适用）**：本 PR 经过几轮 hoare-audit / 哪些维度 reviewed / 哪些 finding 修了哪些 reject 了
- **量化**：N/N pass，关联 PBT properties / property tests / regression locks

### 5. 保证了啥（Guarantees）

正向契约 — 这个 PR 让什么不变式从此恒成立。是 "Risk" 段的对偶面，比 Risk 更主动。

**包含**：
- **不变式（spec → 实现 verified）**：每条 I-N 对应实现位置
- **性质保证**（如"字节正确性"/"行级容错"）
- **数据完整性**（如"单一数据源 / 无 dual-source drift"）
- **已知限制（明示，不掩盖）**：by design 的代价 + 兜底机制

### 6. 风险与回滚（Risk & Rollback）

让 reviewer 一眼判断 "merge 后出问题怎么回滚 / 影响范围多大"。

**包含**：
- **影响面 (blast radius)**：本 PR 改动触达的子系统 / 模块 / 用户路径
- **不向后兼容点**（如 schema 变更 / API 删除 / 文件路径改动）
- **回滚成本**：撤销本 PR 需要做什么（git revert 是否够 / 是否需要数据迁移）
- **监控建议**：merge 后看哪些信号判断没出问题

**重要**：诚实列代价。`tracing::warn!` / `bool flag` / 默认值改动等"看似无害"的改动也要列。

### 7. 考虑过的替代（Alternatives considered）

为什么不是显然的 X 方案。强迫作者**主动**消除反方意见。

**为什么必须写**：
- 防止 reviewer 留言"为什么不用 mutex 而是 mpsc?"
- 历史价值：未来工程师遇到类似问题想用 X 时，可以直接看到当年为什么没选 X
- 配合"奥卡姆剃刀声明"双向闭合：第 3 段说"我们简洁了"，本段说"我们考虑过更复杂的"

**形式**：每条候选方案 1-2 句话 + 拒绝理由。

**反模式**：
- 完全不写本段（"看起来太显然"是错觉，未来读者不一定 share 同一直觉）
- 列一堆稻草人（`use eval()` / `print to disk every emit`）凑数

---

## 元数据约定

- **`Closes #N` / `Fixes #N`**：merge 时自动关 issue
- **PR 标题**：`<type>(<scope>): <短描述>`，type 用 conventional commit (feat/fix/docs/chore/refactor/perf/test/build/ci)
- **scope**：模块 / crate 名

---

## 退化模板（小改动用）

```markdown
## 为啥
（一句话 problem）

## 干了啥
（bullet list）

## 测试
（N/N pass + 关键 case）

Closes #N
```

---

## 模板对比（为什么用这个 7 段而不是别的）

| 主流模板 | 强项 | 缺什么 |
|---|---|---|
| What/Why/How（开源主流） | 简洁 | 无原则 / 无保证 / 无替代方案 |
| Linear (Problem/Solution/Risk/Verification) | 有 Risk | 无原则 / 无保证 |
| ADR (Context/Decision/Consequences/Alternatives) | 有 Alternatives | 无 Testing 段 |
| Checklist (Definition of Done) | 合规友好 | 无 narrative 段 |
| Root Cause (Symptom/Cause/Fix/Why-not-X) | bug fix 专用 | 不适合 feature |

**本规范的取舍**：
- 取 ADR 的 Decision + Alternatives → 第 3 段 Principles + 第 7 段 Alternatives
- 取 Linear 的 Risk + Verification → 第 6 段 Risk & Rollback + 第 4 段 Testing
- 加 ADR / Linear 都没有的 **Guarantees**（第 5 段）—— 正向契约视角
- 砍掉 Checklist 的硬 checkbox（按 PR 性质裁剪 vs 强制套）

---

## 反模式速查

- ❌ "改进了 X" / "优化了 Y" — vague，不可验证
- ❌ 跳过原则段 — 让 reviewer 猜设计意图
- ❌ 跳过替代方案 — reviewer 一定会问，主动写省一轮
- ❌ Risk 段写"无风险" — 任何改动都有 blast radius，诚实列
- ❌ 测试段只写 "全部通过" — 不说测了什么 = 没说
- ❌ 用 prose 段叙述实现细节 — reviewer 读 diff 更快
- ❌ "保证了啥"段写"功能正常" — 写**不变式**不是写**功能**
