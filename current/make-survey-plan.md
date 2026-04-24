# Skill: Make Survey Plan

## Trigger

Trigger when the user asks for a **survey -> organise -> plan** style research plan.
Typical signals: "help me design a survey plan", "how to survey and turn it into a final plan", "this project should start from survey, then ..."

---

## Core Interaction Principles

### a) Backward-chain from the end state, not forward from step one

Start from the final deliverable shape (fields, readability, decision usability), then derive what each stage must produce.
Do not start with a linear "first search, then organize" narrative.

### b) Pseudocode is precise design expression, not implementation code

Pseudocode should express decision logic: what to decide, what thresholds to use, and when to branch.
Do not write implementation details like dict accumulation, set intersection, or field-by-field loops.
Use self-documenting function names. Keep function bodies focused on decision logic.

### c) Do not pre-fill execution-time concrete values

Describe pipeline structure: input types, filtering logic, output schema.
Do not pre-populate keyword lists, concrete search queries, or exhaustive tool names in the plan.

### d) Design precise data structures for each stage

Each stage output must have a clear schema (for example, dataclass style), with fixed slots.
Do not allow free-form text blobs.

### e) Accept and systematize new concepts introduced by the user

When users introduce fuzzy concepts, Claude should structure them, define schemas, and integrate them into the pipeline.
Do not only paraphrase user wording.

### f) Respond to quoted feedback item by item

When users annotate specific plan lines one by one, respond to each item independently with the exact change and effect.
Do not merge or skip any item.

### g) Learn from interruption signals

If the user interrupts output, treat it as a directionality problem.
Stop the current direction, listen to the first interruption sentence, and infer what is missing.

---

## Seven-Stage Framework

### 1. Survey - DFS-style resource discovery

**Design intent**: discover relevant work systematically rather than via random sampling. DFS over citation chains reveals hidden branches that shallow search misses. Per-paper analysis surfaces new directions. Novelty decay gives an objective stop signal.

**Pseudocode**:

```python
def survey(topic, domain_list) -> (list[ResourceReport], CompletenessReport):
    """seed -> dfs_expand -> check_completeness -> return"""
    seeds = generate_initial_seeds(topic)
    reports, skipped = dfs_expand(seeds, topic, domain_list)
    completeness = check_completeness(reports, domain_list)
    return reports, completeness

def dfs_expand(seeds, topic, domain_list) -> (list[ResourceReport], list[SkipRecord]):
    """Priority-queue-driven citation-chain expansion"""
    queue = PriorityQueue(seeds)
    reports, skipped, visited = [], [], set()
    while queue.not_empty() and not is_complete(reports, domain_list):
        url = queue.pop_highest()
        if url in visited:
            continue
        visited.add(url)
        report = analyze_and_filter(url, topic)  # pass only if relevance >= 0.6
        if not report:
            skipped.append(url)
            continue
        reports.append(report)
        for ref in prioritize_refs(report.mentioned_sources):
            if ref.url not in visited:
                queue.add(ref, priority=compute_ref_priority(ref))
        periodically_revisit_skipped(skipped, queue, every=20, sample_rate=0.3)
    return reports, skipped

def prioritize_refs(sources) -> list[SourceRef]:
    """Sort by relation_type: depend/implement > compare > cite"""
    # venue weight > timeliness weight > relevance weight (learned from prior mistakes)
    return sorted(sources, key=lambda s: relation_priority(s.relation_type), reverse=True)

def check_completeness(reports, domain_list) -> CompletenessReport:
    """Three dimensions; stop if 2/3 pass"""
    coverage = all_domains_have_min_2_reports(reports, domain_list)
    novelty = is_saturated(reports, window=20, threshold=0.15)
    citation = top_10_cited_papers_covered_90_percent(reports)
    return CompletenessReport(passed=(sum([coverage, novelty, citation]) >= 2))

@dataclass
class ResourceReport:
    url: str
    title: str
    venue: str                         # conference / journal / arXiv
    year: int
    summary: str                       # <= 200 words
    answer_to_topic: str               # direct answer to core topic question
    mentioned_sources: list[SourceRef]
    source_relations: list[Relation]   # cite | recommend | compare | depend
    gap_notes: str                     # delta from target
    feasibility: str                   # "tooling available" | "theoretical only" | "partially implementable"
    novelty_score: float               # [0,1]
    extra: dict

@dataclass
class SourceRef:
    url: str
    relation_type: str                 # cite | implement | compare | depend
```

**Key decisions**:
- Filtering priority: venue > timeliness > relevance (counterintuitive but validated)
- Every 20 resources, revisit skipped low-priority refs and accept at least 30%
- Keep low-score venue papers if cited by >=3 A-tier papers
- Start with 10-15 high-frequency terms; every 20-30 papers, check for missed directions

**Done criteria**:
- At least 2 of 3 dimensions (coverage / novelty / citation) pass
- All reports conform to `ResourceReport` schema
- Every filtered resource has a skip record (reason + source)
- Use `evo-graph` skill to generate a citation-link graph (see below)

**Citation-link graph**: After Survey, use `evo-graph` skill (`skills/evo-graph.md`) to visualize all `ResourceReport` citation relations in one interactive 2D timeline. Mapping:
- One paper -> one node (`id` = URL slug, `name` = title, `year` = year, `category` = venue or research direction)
- Each relation in `source_relations` -> one edge (`relation` is cite / recommend / compare / depend)
- `refCount` = number of times the paper is cited by collected papers

Output is a single self-contained HTML file in the same directory as survey outputs. The graph should reveal citation paths, high-impact hubs, clusters, and gaps, and feed structural input into Stage 3 method fusion.

---

### 2. Performable Tests - Feasibility validation with minimal context

**Design intent**: dispatch multiple sub-agents with minimal context to expose real learning curves and tooling defects. Core assumption: zero-to-one execution reveals practical obstacles paper analysis misses.

**Pseudocode**:

```python
def run_performable_tests(methods) -> list[SubAgentReport]:
    """filter_candidates -> dispatch_all -> collect_reports"""
    candidates = filter_testable(methods)  # exclude purely theoretical items
    tasks = [dispatch(make_minimal_task(m)) for m in candidates]
    return collect_with_timeout(tasks)

def make_minimal_task(method) -> TaskDescription:
    """Provide only doc entrypoint + one-line goal, no full survey output or paper text"""
    return TaskDescription(
        goal=f"Evaluate viability of {method} from scratch",
        entry_point=method.doc_url,
        output_schema=SubAgentReport,
        time_budget=estimate_effort(method)
    )

def judge_viability(report) -> str:
    """Failure != not successful; failure = cannot produce a clear judgment"""
    if report.produced_clear_judgment:
        return report.viability
    if report.stuck_on_env and elapsed < budget * 0.3:
        return "environment-dependent"
    return "inconclusive"

def collect_with_timeout(tasks) -> list[SubAgentReport]:
    """Record task_id + estimated_duration; mark timeout if elapsed > 1.5x estimate"""
    for task in tasks:
        if task.elapsed > task.estimated * 1.5:
            task.status = "timeout"
    return [t.report for t in tasks]

@dataclass
class SubAgentReport:
    task_id: str
    method_name: str
    elapsed_time: float                 # minutes
    status: str                         # success | fail | timeout
    obstacles: list[str]
    unexpected_findings: list[str]
    viability: str                      # Viable | Needs Modification | Not Viable | environment-dependent
    key_findings: str                   # 3-5 sentences
    resource_consumption: dict
```

**Key decisions**:
- Sub-agents receive minimal context only (doc entrypoint + task description), not full survey output
- Timeout policy: >1.5x estimate -> timeout; if still environment setup after 30% of budget -> environment-dependent
- Record `task_id` + `estimated_duration` on dispatch for tracking
- Require sub-agent acknowledgement of output schema before dispatch

**Done criteria**:
- Every key method tested by at least one sub-agent
- Every method has an explicit viability label (no "uncertain" / "it depends")
- Generate a summary report (viability distribution + shared obstacles + unexpected findings)

---

### 3. Pre-unified Graph - Merge all methods into one graph

**Design intent**: place all methods in one coordinate system so shared structure and divergence are obvious. Graph beats lists/tables: shared steps are reused, divergence points are explicit, and ordering is encoded by edges.

**Pseudocode**:

```python
def build_graph(survey_reports, test_results) -> ValidatedGraph:
    """extract_nodes -> align_steps -> mark_divergences -> validate_topology"""
    methods = extract_all_methods(survey_reports, test_results)
    graph = Graph()
    for method in methods:
        steps = extract_steps(method)
        add_or_merge_nodes(graph, steps, method)  # merge same-named steps into shared nodes
        add_sequence_edges(graph, steps)
    mark_divergences(graph)  # same node with multiple distinct successors = divergence
    validation = validate_topology(graph)
    if not validation.ok:
        graph = auto_repair(graph, validation)
    return ValidatedGraph(graph, validation)

def validate_topology(graph) -> ValidationResult:
    """Four checks: single entry, has exit, no isolated subgraph, all cycles must be iteration/rollback"""
    return ValidationResult(
        single_entry=count_entry_nodes(graph) == 1,
        has_exit=count_exit_nodes(graph) > 0,
        no_isolated=find_isolated_components(graph) == [],
        all_cycles_are_iteration=all(
            edge.type in ("retry", "rollback", "refinement")
            for edge in find_cycle_edges(graph))
    )

def mark_divergences(graph):
    """Node granularity = meaningful decision unit; divergence = choice that materially affects outcome"""
    for node in graph.nodes:
        successors = get_distinct_successors_across_methods(node)
        if len(successors) > 1:
            node.is_divergence = True
            node.options = successors

@dataclass
class GraphNode:
    node_id: str
    name: str
    node_type: str                     # core (shared) | variant (method-specific)
    belongs_to_methods: set[str]
    is_divergence: bool = False
    options: list[str] = None

@dataclass
class ValidationResult:
    single_entry: bool
    has_exit: bool
    no_isolated: bool
    all_cycles_are_iteration: bool
    ok: bool                           # all four checks pass
```

**Key decisions**:
- Node granularity = meaningful decision/execution unit, not each code line or whole method
- Shared steps use solid lines; method-specific steps use color variants; divergence points use diamond shapes
- Iteration cycles (retry/rollback/refinement) are valid; dependency cycles (A->B->A without iteration semantics) are invalid
- For isolated subgraphs: try auto-connect to nearest valid node first; if unreasonable, mark "needs human review"

**Done criteria**:
- Pass all four `validate_topology` checks (or pass after auto-repair)
- All divergence points marked (at least one)
- A reader can understand end-to-end flow in <=2 minutes from the visualization

---

### 4. Organise - Unified schema and indexing

**Design intent**: normalize all materials into one schema and build bidirectional indices. Core assumption: consistent schema + queryable index is the prerequisite for automation.

**Pseudocode**:

```python
def organise(reports, graph) -> OrganisedStructure:
    """validate_all_schemas -> build_indices -> save_structure"""
    validated = [validate_and_fill(r) for r in reports]  # fill empty fields with "N/A"
    forward_index = build_forward_index(graph, validated)  # graph node -> resources
    reverse_index = build_reverse_index(validated, graph)  # resource -> graph nodes
    return OrganisedStructure(validated, forward_index, reverse_index)

def build_reverse_index(reports, graph) -> dict[str, list[str]]:
    """Resource -> graph nodes index for reverse traceability"""
    return {r.url: find_nodes_citing(graph, r) for r in reports}

def validate_and_fill(report) -> ResourceReport:
    """Ensure non-empty fields; fill empties with 'N/A' instead of dropping"""
    for field in ResourceReport.fields():
        if getattr(report, field) is None:
            setattr(report, field, "N/A")
    return report

@dataclass
class OrganisedStructure:
    reports: list[ResourceReport]       # all validated resource reports
    forward_index: dict[str, list[str]] # graph_node_id -> [resource_url]
    reverse_index: dict[str, list[str]] # resource_url -> [graph_node_id]
    classification: dict                # layered by venue/year/method (objective attributes)
```

**Key decisions**:
- Layer by objective attributes (venue / year / method), not subjective themes
- Explicitly mark all empty fields as "N/A"; do not drop fields
- Reverse index is mandatory for fast tracing from any resource back to graph nodes

**Done criteria**:
- All reports validate against schema
- Forward + reverse indices are complete
- Output generated in required format (markdown / json)

---

### 5. Summarise - Resolve divergence point by point

**Design intent**: provide recommendation + rationale for each divergence point in the pre-unified graph. This is not a rewritten survey. Shared steps are not repeated; only divergence decisions are handled.

**Pseudocode**:

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
    """Three trigger cases for human decision"""
    return (
        max_confidence(scores) < 0.7
        or involves_user_preference(divergence)
        or affects_core_field(divergence)
    )

def score_options(options, evidence) -> dict[str, float]:
    """Composite score based on paper count, test pass rate, implementation maturity"""
    return {opt: weighted_score(evidence[opt]) for opt in options}

@dataclass
class DivergenceResolution:
    divergence_id: str
    recommended_option: str | None      # None means human decision required
    confidence: float                    # [0, 1]
    evidence_sources: list[str]          # supporting resource URLs
    reason: str                          # recommendation rationale or deferral rationale
    human_decision_needed: bool
```

**Key decisions**:
- Handle divergence points only; do not repeat shared steps
- If confidence < 0.7 -> mark as human decision required
- If user preference or core fields are involved -> force human confirmation even when confidence is high
- Every recommendation must be traceable to sources (papers / test results)

**Done criteria**:
- Every divergence point has a recommendation + rationale (or explicit human-decision flag)
- Every recommendation is source-traceable
- Produce a human-decision item list

---

### 6. Strategic Report - Non-technical summary for decision makers

**Design intent**: transform technical detail into decision-ready language for stakeholders. This is an independent stage (not a sub-step of Summarise), with a different audience and information architecture.

**Pseudocode**:

```python
def strategic_report(resolutions, organised) -> StrategicReport:
    """translate_findings -> extract_risks -> estimate_effort -> build_comparison"""
    findings = translate_to_nontechnical(resolutions)
    risks = extract_risks_with_probability(resolutions, organised)
    effort = estimate_effort_per_option(resolutions)
    comparison = build_comparison_table(findings, risks, effort)
    return StrategicReport(findings, risks, effort, comparison)

def translate_to_nontechnical(resolutions) -> list[Finding]:
    """Technical conclusions -> business-impact language, with source traceability"""
    return [
        Finding(
            what=r.recommended_option,
            why=r.reason,
            impact=estimate_business_impact(r),
            source=r.evidence_sources,
        )
        for r in resolutions
    ]

def build_comparison_table(findings, risks, effort) -> ComparisonTable:
    """Option comparison table: one row per option with benefit/risk/cost/time"""
    return ComparisonTable(rows=zip_into_rows(findings, risks, effort))

@dataclass
class StrategicReport:
    executive_summary: str              # 2-3 pages, non-technical language
    findings: list[Finding]
    risks: list[Risk]                   # each includes probability + impact estimate
    effort_estimates: dict              # option -> person-days
    comparison_table: ComparisonTable   # benefit / risk / cost / time
    source_traceability: dict           # data point -> source URL
```

**Key decisions**:
- Non-technical audience: use concrete numbers ("40% faster"), not vague wording ("significantly faster")
- Every data point must be source-traceable
- This is a standalone stage, not a byproduct of Summarise
- Keep executive summary to 2-3 pages

**Done criteria**:
- Non-technical reader can understand core recommendation within 15 minutes
- Every data point has source traceability
- Include option comparison table (benefit / risk / cost / time)

---

### 7. Ultimate Plan - Executable steps + quantified human intervention

**Design intent**: produce an executable final plan from the recommended path in the graph. Core assumption: success criteria and intervention rules must be machine-judgable condition expressions, not vague slogans.

**Pseudocode**:

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
            source_trace=trace_to_evidence(node, resolutions),
        )
        steps.append(step)
    interaction_protocol = derive_interaction_protocol(steps)
    return steps, interaction_protocol

def derive_intervention_rule(node) -> str:
    """Write human intervention as 'when X do Y else Z', never 'intervene if needed'"""
    return f"When {node.trigger}, do {node.human_action}; otherwise, {node.auto_action}"

def derive_interaction_protocol(steps) -> InteractionProtocol:
    """Three interaction categories: clarification triggers, confirmation points, termination conditions"""
    return InteractionProtocol(
        clarifications=[s for s in steps if s.has_ambiguity],
        confirmations=[s for s in steps if s.is_critical],
        terminations=[s for s in steps if s.can_fail_fatally],
    )

@dataclass
class ExecutionStep:
    action: str
    input_type: str
    output_type: str
    success_criterion: str              # testable condition, not "try best"
    intervention: str                   # must follow "when X do Y else Z"
    fallback: str                       # backup path on failure
    source_trace: list[str]             # source papers / test results

@dataclass
class InteractionProtocol:
    clarifications: list[str]           # when clarification is needed
    confirmations: list[str]            # when confirmation is required
    terminations: list[str]             # when execution should terminate
```

**Key decisions**:
- Write intervention as "when X do Y else Z", never "manual intervention when needed"
- Every step must have machine-checkable success criteria; ban fuzzy terms like "as much as possible" or "as soon as possible"
- >= 95% of steps should be source-traceable to papers or test results
- Interaction protocol must define clarification triggers, confirmation points, and termination conditions

**Done criteria**:
- Every step has input/output types + success criteria + intervention rule + fallback
- All human intervention points have explicit conditionals ("when X do Y else Z")
- >= 95% of steps are source-traceable
- Provide execution milestones + verification report template

---

## Output Quality Standards

- **High efficiency to understand**: readers can grasp the whole quickly (unified templates + graph navigation)
- **Easy to verify**: every conclusion traces to a source (`source` field in reports; recommendations cite supporting reports)
- **Minimal human intervention**: quantified rules, not slogans (condition + auto/manual branches, not "try to automate")
- **Algorithm-level expression**: the plan itself uses pseudocode (pipeline function + dataclass for each stage)

---

## Anti-Patterns

| Anti-pattern | Consequence |
|--------------|-------------|
| Natural language only, no pseudocode | Not executable; inconsistent interpretation |
| Pre-filling concrete execution values in the plan (keyword lists, etc.) | Plan becomes execution log, not reusable |
| Linear method list instead of merged graph | Method relationships are hidden; recommendation decisions are weak |
| Summarise rewritten as full survey recap | Redundant output; high cognitive load |
| "minimal human intervention" used as slogan only | Unclear manual decision timing during execution |
| Merging or skipping user feedback items | Issues remain and reappear in next revision |
| Not recording relation types between sources | Weak structure for later synthesis and automation |
| Pseudocode written as full implementation (dict accumulation, field loops) | Noise hides design decisions; maintenance cost rises |
| Topology checks banning all cycles | Valid iteration/rollback loops are incorrectly removed; only dependency cycles should be banned |

---

## Format and Storage

- Write the plan in the user's language (usually Chinese), while keeping technical terms in English when appropriate
- Use Python-style pseudocode unless the user has explicitly demonstrated another language
- Save to `~/.claude/plans/<slug>.md`, where `slug` is generated from topic keywords
