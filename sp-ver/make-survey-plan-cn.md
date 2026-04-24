# Skill: Make Survey Plan

## Trigger

当用户要求制定 **survey -> organise -> plan** 类型的调研计划时触发。
典型信号："帮我制定调研计划"、"survey 怎么做，最终方案怎么形成"、"这个项目要先 survey，然后……"

---

## 核心交互原则

### a) 从终态倒推，不要从起点正推

先讨论最终产出物的形态（包含哪些字段、人怎么读懂它），然后倒推每个阶段必须产出什么。
不要从"第一步搜索，第二步整理"顺序展开。

### b) 伪代码是精确的设计表达，不是实现代码

伪代码表达"做什么决策、用什么阈值、在什么条件下选择哪条路"。
不要写 dict 累加、set intersection、for loop 遍历字段等实现细节。
函数名自文档化，函数体只保留决策逻辑。

### c) 不预写执行时才知道的具体值

描述 pipeline 的结构——输入类型、过滤条件逻辑、输出 schema。
不要在计划中填写具体关键词列表、搜索查询、工具名称穷举。

### d) 为每个阶段设计精确的数据结构

每阶段输出有明确的 schema（dataclass），固定槽位。
不要让输出是自由文本 blob。

### e) 接受并系统化用户引入的新概念

用户提出模糊概念时，Claude 负责填充结构、设计 schema、融入整体 pipeline。
不要只做简单转述。

### f) 逐条响应引用+批注式反馈

用户引用计划中的具体句子逐条批注时，每条都单独响应，明确说明修改方式和效果。
不要合并或跳过任何一条。

### g) 从中断信号学习

用户在输出过程中中断 = 方向性问题。停止当前方向，听完用户第一句话，从中倒推缺失的是什么。

---

## 七阶段框架

### 1. Survey — DFS 展开式资源发现

**设计意图**：系统性地发现所有相关工作，而非随机抽样。DFS 追踪引用链能挖出表面搜索找不到的方向，逐篇分析能发现新研究方向，新信息增量自然衰减提供客观停止信号。

**伪代码**：

```python
def survey(topic, domain_list) -> (list[ResourceReport], CompletenessReport):
    """seed -> dfs_expand -> check_completeness -> return"""
    seeds = generate_initial_seeds(topic)
    reports, skipped = dfs_expand(seeds, topic, domain_list)
    completeness = check_completeness(reports, domain_list)
    return reports, completeness

def dfs_expand(seeds, topic, domain_list) -> (list[ResourceReport], list[SkipRecord]):
    """优先队列驱动的引用链展开"""
    queue = PriorityQueue(seeds)
    reports, skipped, visited = [], [], set()
    while queue.not_empty() and not is_complete(reports, domain_list):
        url = queue.pop_highest()
        if url in visited: continue
        visited.add(url)
        report = analyze_and_filter(url, topic)  # relevance >= 0.6 才通过
        if not report: skipped.append(url); continue
        reports.append(report)
        for ref in prioritize_refs(report.mentioned_sources):
            if ref.url not in visited:
                queue.add(ref, priority=compute_ref_priority(ref))
        periodically_revisit_skipped(skipped, queue, every=20, sample_rate=0.3)
    return reports, skipped

def prioritize_refs(sources) -> list[SourceRef]:
    """按 relation_type 排序：depend/implement > compare > cite"""
    # venue 权重 > timeliness 权重 > relevance 权重（经验教训：反转顺序）
    return sorted(sources, key=lambda s: relation_priority(s.relation_type), reverse=True)

def check_completeness(reports, domain_list) -> CompletenessReport:
    """三维度判断，2/3 通过即可停止"""
    coverage = all_domains_have_min_2_reports(reports, domain_list)
    novelty = is_saturated(reports, window=20, threshold=0.15)
    citation = top_10_cited_papers_covered_90_percent(reports)
    return CompletenessReport(passed=(sum([coverage, novelty, citation]) >= 2))

@dataclass
class ResourceReport:
    url: str
    title: str
    venue: str                         # 会议/期刊/arXiv
    year: int
    summary: str                       # 200 字以内
    answer_to_topic: str               # 对核心问题的直接回答
    mentioned_sources: list[SourceRef]
    source_relations: list[Relation]   # cite | recommend | compare | depend
    gap_notes: str                     # 与目标的 delta
    feasibility: str                   # "有工具实现" | "理论描述" | "部分可实现"
    novelty_score: float               # [0,1]
    extra: dict

@dataclass
class SourceRef:
    url: str
    relation_type: str                 # cite | implement | compare | depend
```

**关键决策**：
- 过滤优先级：venue > timeliness > relevance（经验教训：反转常规直觉顺序）
- 每 20 篇资源回溯被跳过的低优先级引用，至少采纳 30%
- 低分 venue 论文若被 >=3 篇 A 类论文引用，仍保留
- 初始 10-15 个高频词，每发现 20-30 篇后回顾是否遗漏新方向

**完成标准**：
- 三维度（coverage / novelty / citation）中 >=2 个通过
- 所有 report 符合 ResourceReport schema
- 被过滤资源均有 skip 记录（理由 + 来源）
- 使用 `evo-graph` skill 生成文献引用勾连图（见下）

**文献引用勾连图**：Survey 完成后，用 `evo-graph` skill（`skills/evo-graph.md`）将所有 ResourceReport 的引用关系可视化为一张交互式 2D 时间线图。具体映射：
- 每篇文献 → 一个节点（`id`=url slug, `name`=title, `year`=year, `category`=venue 或研究方向）
- `source_relations` 中的每条 Relation → 一条连线（`relation` 取 cite / recommend / compare / depend）
- `refCount` = 该文献被其他已收集文献引用的次数

产出物为单个自包含 HTML 文件，保存到与 survey 输出同目录下。该图用于直观展示文献间的引用链路、识别高影响力枢纽论文、发现引用簇和断裂带，为后续 Stage 3 的方法融合提供结构性输入。

---

### 2. Performable Tests — 最少 context 验证可行性

**设计意图**：派多个 sub-agent 用最少 context 做实战测试，暴露真实学习曲线和工具缺陷。核心假设：初学者从零探索能发现纸上谈兵找不到的坑。

**伪代码**：

```python
def run_performable_tests(methods) -> list[SubAgentReport]:
    """filter_candidates -> dispatch_all -> collect_reports"""
    candidates = filter_testable(methods)  # 排除纯理论无法实测的
    tasks = [dispatch(make_minimal_task(m)) for m in candidates]
    return collect_with_timeout(tasks)

def make_minimal_task(method) -> TaskDescription:
    """只给文档入口 + 一句话任务描述，不给 survey 结果或论文全文"""
    return TaskDescription(
        goal=f"从零评估 {method} 的可行性",
        entry_point=method.doc_url,
        output_schema=SubAgentReport,
        time_budget=estimate_effort(method)
    )

def judge_viability(report) -> str:
    """失败 != 没成功；失败 = 无法产出清晰判断"""
    if report.produced_clear_judgment: return report.viability
    if report.stuck_on_env and elapsed < budget * 0.3: return "environment-dependent"
    return "inconclusive"

def collect_with_timeout(tasks) -> list[SubAgentReport]:
    """记录 task_id + estimated_duration；超时 1.5x 标记为 timeout"""
    for task in tasks:
        if task.elapsed > task.estimated * 1.5: task.status = "timeout"
    return [t.report for t in tasks]

@dataclass
class SubAgentReport:
    task_id: str
    method_name: str
    elapsed_time: float                 # 分钟
    status: str                         # success | fail | timeout
    obstacles: list[str]
    unexpected_findings: list[str]
    viability: str                      # Viable | Needs Modification | Not Viable | environment-dependent
    key_findings: str                   # 3-5 句
    resource_consumption: dict
```

**关键决策**：
- Sub-agent 只拿最少 context（文档入口 + 任务描述），不给 survey 全文
- 超时处理：超过预估 1.5x 标记为 timeout，超 30% 时间仍在搭环境标记为 environment-dependent
- 派遣时记录 task_id + estimated_duration，用于状态追踪
- 派遣前要求 sub-agent acknowledge 输出 schema

**完成标准**：
- 所有关键方法至少派 1 个 sub-agent 测试
- 每个方法有明确 viability 标签（无"不确定"或"看情况"）
- 生成汇总报告（viability 分布 + 共同障碍 + 意外发现）

---

### 3. Pre-unified Graph — 融合所有方案成一张图

**设计意图**：将所有方法放在同一坐标系下，使人一眼看出共性和分歧。图优于表格/列表：共性步骤复用、分歧点显式标注、执行顺序通过边表达。

**伪代码**：

```python
def build_graph(survey_reports, test_results) -> ValidatedGraph:
    """extract_nodes -> align_steps -> mark_divergences -> validate_topology"""
    methods = extract_all_methods(survey_reports, test_results)
    graph = Graph()
    for method in methods:
        steps = extract_steps(method)
        add_or_merge_nodes(graph, steps, method)  # 同名步骤合并为共有节点
        add_sequence_edges(graph, steps)
    mark_divergences(graph)  # 同一节点有多个不同后继 = 分歧点
    validation = validate_topology(graph)
    if not validation.ok: graph = auto_repair(graph, validation)
    return ValidatedGraph(graph, validation)

def validate_topology(graph) -> ValidationResult:
    """四项检查：唯一入口、有出口、无孤立子图、所有环必须是迭代/回退边"""
    return ValidationResult(
        single_entry=count_entry_nodes(graph) == 1,
        has_exit=count_exit_nodes(graph) > 0,
        no_isolated=find_isolated_components(graph) == [],
        all_cycles_are_iteration=all(
            edge.type in ("retry", "rollback", "refinement")
            for edge in find_cycle_edges(graph))
    )

def mark_divergences(graph):
    """节点粒度 = 有意义的决策点；分歧 = 显著影响结果的选择"""
    for node in graph.nodes:
        successors = get_distinct_successors_across_methods(node)
        if len(successors) > 1:
            node.is_divergence = True
            node.options = successors

@dataclass
class GraphNode:
    node_id: str
    name: str
    node_type: str                     # core (共有) | variant (特有)
    belongs_to_methods: set[str]
    is_divergence: bool = False
    options: list[str] = None

@dataclass
class ValidationResult:
    single_entry: bool
    has_exit: bool
    no_isolated: bool
    all_cycles_are_iteration: bool
    ok: bool                           # 四项全通过
```

**关键决策**：
- 节点粒度 = 一个有意义的决策点或执行单位，不是每行代码或整个方法
- 共有步骤用实线，方案特有用不同颜色，分歧点用钻石形
- 迭代环（retry/rollback/refinement）是合法的；依赖环（A→B→A 无迭代语义）是非法的
- 孤立子图先尝试自动连接到最近节点，连接不合理则标注"需人工审核"

**完成标准**：
- 通过 validate_topology 四项检查（或自动修复后通过）
- 所有分歧点已标注（至少 1 个）
- 可视化后人能在 2 分钟内理解整体流程

---

### 4. Organise — 统一 schema 与索引

**设计意图**：将所有材料按统一模板整理，建立双向索引。核心假设：一致的 schema + 可查询的索引是自动化处理的前提。

**伪代码**：

```python
def organise(reports, graph) -> OrganisedStructure:
    """validate_all_schemas -> build_indices -> save_structure"""
    validated = [validate_and_fill(r) for r in reports]  # 空字段标 "N/A"
    forward_index = build_forward_index(graph, validated)  # graph节点 -> 资源列表
    reverse_index = build_reverse_index(validated, graph)  # 资源 -> graph节点列表
    return OrganisedStructure(validated, forward_index, reverse_index)

def build_reverse_index(reports, graph) -> dict[str, list[str]]:
    """资源 -> graph 节点的反向索引，支持从任一资源追溯到图中位置"""
    return {r.url: find_nodes_citing(graph, r) for r in reports}

def validate_and_fill(report) -> ResourceReport:
    """确保所有字段非空；空字段标 'N/A'，不删除"""
    for field in ResourceReport.fields():
        if getattr(report, field) is None: setattr(report, field, "N/A")
    return report

@dataclass
class OrganisedStructure:
    reports: list[ResourceReport]       # 所有已验证的资源报告
    forward_index: dict[str, list[str]] # graph_node_id -> [resource_url]
    reverse_index: dict[str, list[str]] # resource_url -> [graph_node_id]
    classification: dict                # 按 venue/year/method 分层（客观属性，非主观主题）
```

**关键决策**：
- 按客观属性分层（venue / year / method），不按主观主题分类
- 所有空字段显式标为 "N/A"，不删除
- 反向索引是必须的：支持从任一资源快速定位到 graph 中的对应节点

**完成标准**：
- 所有 report 符合 schema 且通过验证
- 正向 + 反向索引完整建立
- 按指定格式生成输出（markdown / json）

---

### 5. Summarise — 逐分歧解决

**设计意图**：针对 pre-unified graph 中的每个分歧点给出推荐选择及理由。不是重写综述——共性步骤不重复描述，只处理分歧决策点。

**伪代码**：

```python
def summarise(graph, organised) -> list[DivergenceResolution]:
    """for each divergence -> gather_evidence -> score_options -> recommend_or_defer"""
    resolutions = []
    for div in graph.divergence_points:
        evidence = gather_evidence(div, organised)
        scores = score_options(div.options, evidence)
        if requires_human_decision(scores, div):
            resolutions.append(defer_to_human(div, scores, reason="confidence < 0.7"))
        else:
            resolutions.append(recommend(div, best_option(scores), evidence))
    return resolutions

def requires_human_decision(scores, divergence) -> bool:
    """三种情况触发人工决策"""
    return (max_confidence(scores) < 0.7
            or involves_user_preference(divergence)
            or affects_core_field(divergence))

def score_options(options, evidence) -> dict[str, float]:
    """基于论文数量、测试通过率、实现成熟度综合评分"""
    return {opt: weighted_score(evidence[opt]) for opt in options}

@dataclass
class DivergenceResolution:
    divergence_id: str
    recommended_option: str | None      # None = 需人工决策
    confidence: float                   # [0, 1]
    evidence_sources: list[str]         # 支持该推荐的资源 URL
    reason: str                         # 推荐理由或人工决策原因
    human_decision_needed: bool
```

**关键决策**：
- 只处理分歧点，共性步骤不重复描述
- confidence < 0.7 时 → 标记为需人工决策
- 涉及用户偏好或核心字段时 → 即使 confidence 高也触发人工确认
- 每个推荐必须追溯到来源（论文 / 测试结果）

**完成标准**：
- 所有分歧点都有推荐选项 + 理由（或明确标记为需人工决策）
- 每个推荐可追溯到来源
- 生成人工决策项列表

---

### 6. Strategic Report — 面向决策者的非技术总结

**设计意图**：将技术细节转化为决策者可理解的总结。这是独立阶段（不是 Summarise 的子步骤），目标读者和信息组织方式完全不同。

**伪代码**：

```python
def strategic_report(resolutions, organised) -> StrategicReport:
    """translate_findings -> extract_risks -> estimate_effort -> build_comparison"""
    findings = translate_to_nontechnical(resolutions)
    risks = extract_risks_with_probability(resolutions, organised)
    effort = estimate_effort_per_option(resolutions)
    comparison = build_comparison_table(findings, risks, effort)
    return StrategicReport(findings, risks, effort, comparison)

def translate_to_nontechnical(resolutions) -> list[Finding]:
    """技术结论 -> 业务影响描述，所有数据可追溯到来源"""
    return [Finding(what=r.recommended_option, why=r.reason,
                    impact=estimate_business_impact(r), source=r.evidence_sources)
            for r in resolutions]

def build_comparison_table(findings, risks, effort) -> ComparisonTable:
    """方案对比表：每个选项的收益/风险/成本/时间一行"""
    return ComparisonTable(rows=zip_into_rows(findings, risks, effort))

@dataclass
class StrategicReport:
    executive_summary: str              # 2-3 页，非技术语言
    findings: list[Finding]
    risks: list[Risk]                   # 每个有概率 + 影响评估
    effort_estimates: dict              # option -> person-days
    comparison_table: ComparisonTable   # 收益/风险/成本/时间
    source_traceability: dict           # 数据点 -> 来源 URL
```

**关键决策**：
- 面向非技术读者：所有指标用具体数值（"40% 更快"而非"显著更快"）
- 所有数据点可追溯到来源；是独立阶段，不是 Summarise 的附属产出
- Executive summary 控制在 2-3 页

**完成标准**：
- 非技术读者能在 15 分钟内理解核心建议
- 每个数据都有来源追溯
- 包含方案对比表（收益 / 风险 / 成本 / 时间）

---

### 7. Ultimate Plan — 可执行步骤 + 量化人工干预

**设计意图**：从 graph 的推荐路径生成最终可执行方案。核心假设：成功条件和人工干预规则必须是可机器判断的条件表达式，不是模糊口号。

**伪代码**：

```python
def ultimate_plan(graph, resolutions) -> list[ExecutionStep]:
    """for each recommended_node -> make_step(success_criterion, intervention_rule, fallback)"""
    path = extract_recommended_path(graph, resolutions)
    steps = []
    for node in path:
        step = ExecutionStep(
            action=node.name,
            input_type=infer_input(node),
            output_type=infer_output(node),
            success_criterion=derive_success_criterion(node),
            intervention=derive_intervention_rule(node),
            fallback=derive_fallback(node),
            source_trace=trace_to_evidence(node, resolutions)
        )
        steps.append(step)
    interaction_protocol = derive_interaction_protocol(steps)
    return steps, interaction_protocol

def derive_intervention_rule(node) -> str:
    """人工干预写 '当X时做Y否则Z'，不写 '必要时人工介入'"""
    return f"当 {node.trigger} 时 {node.human_action}，否则 {node.auto_action}"

def derive_interaction_protocol(steps) -> InteractionProtocol:
    """三类交互点：澄清触发、确认点、终止条件"""
    return InteractionProtocol(
        clarifications=[s for s in steps if s.has_ambiguity],
        confirmations=[s for s in steps if s.is_critical],
        terminations=[s for s in steps if s.can_fail_fatally])

@dataclass
class ExecutionStep:
    action: str
    input_type: str
    output_type: str
    success_criterion: str              # 可判断的条件，非 "尽量"
    intervention: str                   # "当X时做Y否则Z" 格式
    fallback: str                       # 失败时的备选
    source_trace: list[str]             # 来源论文/测试结果

@dataclass
class InteractionProtocol:
    clarifications: list[str]           # 何时需要澄清
    confirmations: list[str]            # 何时需要确认
    terminations: list[str]             # 何时应终止
```

**关键决策**：
- 人工干预写 "当X时做Y否则Z"，不写 "必要时人工介入"
- 每个步骤有可判断的成功条件，禁用 "尽量" "尽快"
- >= 95% 步骤能追溯到来源论文或测试结果
- 交互协议定义澄清触发、确认点、终止条件

**完成标准**：
- 每个步骤有输入/输出类型 + 成功条件 + 干预规则 + 备选方案
- 所有人工介入点有明确条件（"当X时做Y否则Z"）
- >= 95% 步骤可追溯到来源
- 生成 execution milestones + verification report 模板

---

## 产出质量标准

- **高效 to understand**：读者不需从头读完就能抓住全貌（统一模板 + graph 导航）
- **Easy to verify**：每个结论追溯到来源（report 有 source 字段，推荐注明依据的 report）
- **Minimal human intervention**：量化规则而非口号（判断条件 + 自动/人工分支，不写"尽量自动"）
- **算法级描述**：plan 本身用伪代码（每阶段有 pipeline 函数 + dataclass）

---

## 反模式

| 反模式 | 后果 |
|--------|------|
| 只有自然语言没有伪代码 | 无法执行，各人理解不同 |
| 计划中预写具体执行值（关键词列表等） | 计划变执行记录，不可复用 |
| 线性列举方法而不是融合成图 | 看不出方法间关系，无法做推荐决策 |
| summarise 变成重写综述 | 产出冗余，读者负担大 |
| "minimal human intervention" 只是口号 | 执行时仍不清楚何时人工介入 |
| 合并或跳过用户的某一条反馈 | 问题残留，下一轮仍被指出 |
| 不记录 source 之间的关系类型 | 后续综合缺乏结构，难以自动化 |
| 伪代码写成完整实现（dict 累加、for loop 遍历字段） | 噪音掩盖设计决策，维护成本高 |
| 拓扑检查禁止所有环 | 合法的迭代/回退环被误杀；应只禁止依赖环 |

---

## 格式与存储

- 用用户使用的语言写 plan（通常中文，技术术语保留英文）
- 伪代码用 Python 风格（除非用户示范了其他语言）
- 保存到 `~/.claude/plans/<slug>.md`，slug 由主题关键词生成
