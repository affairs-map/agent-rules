# CONTEXT_MANAGER Agent Ruleset

**Role**: Context window budgeting, pruning, and memory management  
**Team**: Observability & Traceability (F-2)  
**Primary Dependency**: TRACER, HANDOFF_AUDITOR  
**Start Event**: Phase 0 T+0:00  
**Stop Event**: Phase 3 complete

---

## Core Mandate

Monitor context window utilization across all agents. Enforce explicit budgets per agent. Trigger summarization/pruning before context limits to prevent cost explosion and hallucination from overload.

**Success Metric**: All agents operate within 75% of context window; zero context-overload hallucinations; cost reduction of 30%+ vs unbounded approach.

---

## Context Window Budgets

### 1. Per-Agent Allocation

Assign fixed context budgets based on agent role and complexity:

| Agent | Role | Budget | Justification |
|-------|------|--------|----------------|
| DEV_DATA | Entity ingestion | 100K tokens | Needs full schema, rulesets, phase 0 context |
| DEV_UI | UI rendering | 80K tokens | Mockups + design tokens + validation results |
| DEV_MONITOR | Monitoring setup | 40K tokens | SLA rules + threshold definitions |
| DEV_EXECUTOR | Deployment | 60K tokens | Build scripts, rollback plans, monitoring setup |
| QA_DATA | Entity validation | 100K tokens | Full entity store schema + all entity data |
| QA_VISUAL | UI validation | 80K tokens | Design token reference + mockup specs |
| QA_GATE | Final approval | 120K tokens | All QA results + judge council scores |
| CURATOR_TL | Curation lead | 50K tokens | Source authentication rules + evidence |
| CURATOR_SOURCE | Source ops | 60K tokens | Individual source credentials, API configs |

**Total Budget**: ~650K tokens for 9 core agents.

**Why Budgets Matter**:
- Unbounded context causes cost explosion (120K → 200K+ tokens per agent)
- Large contexts introduce hallucination from dilution (model confuses relevant vs irrelevant context)
- Budgets force prioritization: what context matters most for this agent's work?

---

## Pruning Strategy

### 2. Context Windowing Timeline

**Phase 0 Execution (T+0:00 to T+1:30)**
- All agents operate within budget allocation
- Context contains: spawn packet + phase 0 data
- Size: ~40K tokens per agent

**Phase 0→1 Transition (T+1:30 to T+2:00)**
- CONTEXT_MANAGER action: Summarize Phase 0 logs
- **Pruning Rule**: Remove verbose tool call traces, keep only failures + key decisions
- **Summarization Rule**: "Phase 0 entity ingestion: 150 entities loaded, 5 validation failures (listed below), 0 schema errors"
- **Output**: 1-page summary replacing 40K token Phase 0 logs
- **Size Reduction**: 40K → 5K tokens recovered

**Phase 1 Execution (T+1:30 to T+3:00)**
- Agents operate with Phase 0 summary + Phase 1 live context
- Context contains: Phase 0 summary (5K) + Phase 1 data (35K) = 40K tokens
- **Removed**: Verbose Phase 0 traces, redundant validation logs

**Phase 1→2 Transition (T+3:00 to T+3:30)**
- CONTEXT_MANAGER action: Summarize Phase 1 logs
- **Pruning Rule**: Remove intermediate UI mockup iterations, keep only final approved versions
- **Summarization Rule**: "Phase 1 UI rendering: 45 mockups generated, 12 design iterations, final version approved by QA_VISUAL"
- **Output**: 1-page summary replacing Phase 1 artifact logs
- **Size Reduction**: Another 5K tokens recovered

**Phase 2 Execution (T+3:00 to T+4:30)**
- Agents operate with Phase 0 + Phase 1 summaries + Phase 2 live context
- Context: 5K + 5K + 35K = 45K tokens
- Cumulative budget buffer maintained

**Phase 2→3 Transition (T+4:30 to T+5:00)**
- CONTEXT_MANAGER action: Archive Phase 2 monitoring setup rules
- **Pruning Rule**: SLA thresholds stay, verbose metric definitions pruned
- **Output**: Compact SLA table (5K instead of 15K tokens)

**Phase 3 Execution (T+4:30 to T+5:15)**
- Agents operate with all prior summaries + Phase 3 deployment context
- Context: 5K + 5K + 5K + 40K = 55K tokens (well within budget)

---

## Summarization Rules

### 3. Verbose Context → Summary Transformation

**Rule 1: Tool Call Logs**

Before (verbose):
```
{
  "tool": "entity_store.py",
  "call": "create_entity",
  "params": {type: "Person", name: "Rajesh Kumar", ...},
  "result": "success",
  "duration_ms": 250
}
[repeated 150 times in full detail]
```

After (summary):
```
Phase 0 entity creation: 150 entities created successfully via entity_store.py
- 20 Person entities
- 80 Position entities
- 50 Organization entities
No errors, avg duration 250ms per entity
```

Savings: ~40K tokens → 500 tokens

**Rule 2: Validation Result Logs**

Before (verbose):
```
entity_id: pos_001
validation_check_1: PASS
validation_check_2: PASS
validation_check_3: PASS
...
[repeated for all criteria]
```

After (summary):
```
Phase 0 validation: 150/150 entities passed schema validation
Failure summary: None
Critical findings: None
```

Savings: ~15K tokens → 200 tokens

**Rule 3: Design Iteration History**

Before (verbose):
```
Iteration 1: Color #FF0000, received feedback "too red"
Iteration 2: Color #EE0000, received feedback "slightly better"
Iteration 3: Color #DD0000, received feedback "still too bright"
...
Iteration 12: Color #990000, approved by QA_VISUAL
```

After (summary):
```
Phase 1 UI mockups: 45 designs completed over 12 iterations
Final design approved by QA_VISUAL
Color palette finalized: primary=#990000, secondary=#CCCCCC
```

Savings: ~8K tokens → 300 tokens

---

## Pruning Triggers

### 4. Automatic Pruning Conditions

CONTEXT_MANAGER monitors context size per agent and triggers pruning if:

**Trigger 1: Budget Threshold Breach**
```
IF agent_context_size > (budget_allocation * 0.75):
  SUMMARIZE prior phase
  PRUNE verbose logs
  ALERT Udhay
```

Example: DEV_UI budget = 80K, current size = 65K (81%)
- Action: Prune Phase 0 logs immediately
- Result: Context drops to 60K (75% of budget)

**Trigger 2: Phase Boundary**
```
At each phase boundary (T+90, T+150, T+210):
  CONTEXT_MANAGER summarizes completed phase
  Removes verbose logs
  Packs results into 1-page summary
```

**Trigger 3: Cost Alert**
```
IF cumulative_tokens_used > (session_budget * 0.80):
  CONTEXT_MANAGER triggers aggressive pruning
  Removes optional context
  ALERTS Udhay
```

---

## What to Prune vs What to Keep

### 5. Retention Matrix

| Context Type | Keep | Prune | Rationale |
|--------------|------|-------|-----------|
| Tool call traces | Errors only | Success/verbose | Errors needed for debugging, success logs are verbose |
| Validation results | Failed criteria + fixes | Passed criteria logs | Failures inform recovery, passes are noise |
| UI mockup iterations | Final version | All iterations | Final design is what matters, iterations are history |
| Entity creation logs | Entity count summary | Individual entity logs | Count matters, individual logs are redundant |
| Approval signatures | All signatures | Approval reasoning | Who approved matters, reasoning is captured elsewhere |
| Schema definitions | Current version | Deprecated versions | Only current schema is active |
| Performance metrics | Thresholds hit | All individual metrics | What was slow matters, every data point is noise |

---

## Context Pruning Implementation

### 6. Summarization Protocol

When CONTEXT_MANAGER prunes a phase, it creates a **Phase Summary Document**:

```markdown
# Phase [N] Summary
**Duration**: T+[start] to T+[end]  
**Status**: Complete / In Progress

## Entities
- Created: 150
- Validated: 150 passed, 0 failed
- Schema version: 3.2.1

## Artifacts
- Mockups: 45 designs final-approved
- Validation results: All critical items resolved
- Issues: None outstanding

## Key Decisions
- RBI Governor successor chain depth: 10 levels (consensus with aspirant)
- Scheme disambiguation groups: 30 schemes mapped

## Handoff Status
- Ready for Phase [N+1]: YES
- Missing dependencies: None
- Constraints understood by next team: YES

## Metrics
- Execution time: [X] minutes
- Budget utilization: [Y]%
- Cost: [Z] tokens
```

This 1-page document replaces 40-50K tokens of verbose logs.

---

## Context Tracking Dashboard

### 7. Observability

CONTEXT_MANAGER maintains real-time dashboard:

```
CONTEXT_MANAGER STATUS (T+2:45)

Agent Budgets:
  DEV_DATA:  45K / 100K (45%)  ✓ Within budget
  DEV_UI:    62K / 80K  (78%)  ⚠ Approaching limit
  QA_DATA:   88K / 100K (88%)  ⚠ PRUNE RECOMMENDED
  QA_GATE:   110K / 120K (92%) ⚠ URGENT PRUNE

Phase Summary Sizes:
  Phase 0 summary: 5K tokens (compressed from 40K)
  Phase 1 summary: 8K tokens (ongoing compression)

Total Session:  ~580K / 650K (89%)
  Pruning Headroom: 70K tokens remaining

Last Prune Event: T+2:00 (Phase 0→1 transition)
Next Scheduled: T+3:00 (Phase 1→2 transition)

Recommendation: Proceed with current phase, prune at next boundary
```

---

## Recovery from Context Exhaustion

### 8. Emergency Pruning

If an agent approaches 95% of budget (critical), CONTEXT_MANAGER:

1. **Immediate Action**: Prune all non-critical context
   - Remove Phase N-2 detailed logs
   - Keep only Phase N-1 summary and Phase N active data
   - Result: Context drops 10-15K tokens

2. **Escalation**: Report to ERROR_MONITOR
   - May indicate underestimated budget
   - May indicate excessive data generation by prior phase
   - DEBUGGER_CONTEXT spawned to investigate

3. **Prevention**: Adjust budget for future runs
   - If DEV_DATA regularly hits 90K+ on 100K budget, increase to 120K next time
   - If no agent ever exceeds 50K, decrease budget and reallocate

---

## Token Accounting

### 9. Budget Tracking

CONTEXT_MANAGER tracks every token:

```
Phase 0 Execution:
  DEV_DATA spawn:      500 tokens
  DEV_DATA execution:  40K tokens
  DEV_UI spawn:        400 tokens
  DEV_UI execution:    35K tokens
  Subtotal Phase 0:    ~76K tokens

Phase 0→1 Pruning:
  Phase 0 logs summary: 5K tokens (saved 35K)
  Pruned context:      -35K tokens
  Net after pruning:   ~46K tokens

[continues for all phases]
```

This accounting shows:
- Where tokens are spent (which agents, which phases)
- Effectiveness of pruning (40K → 5K = 87.5% reduction)
- Cumulative burn rate (is session on track to use full allocation?)

---

## Compliance Checklist

- [ ] All agents assigned context budgets
- [ ] Budgets reviewed and approved by Udhay
- [ ] Monitoring dashboard real-time updated
- [ ] Pruning triggers automatically firing at phase boundaries
- [ ] Phase summaries generated and replacing verbose logs
- [ ] Context size never exceeds 75% of agent budget during normal operation
- [ ] Emergency pruning procedures tested
- [ ] Token accounting complete and auditable
- [ ] Cost reduction achieved (target: 30%+ savings vs unbounded)
- [ ] No hallucination from context overload detected

---

## Integration Points

**Receives From**:
- TRACER (context size metrics)
- HANDOFF_AUDITOR (packet sizes)
- All agents (context utilization reports)

**Sends To**:
- All agents (pruning directives, updated context)
- God log (budget compliance records)
- Udhay (context dashboard, budget alerts)

---

## Termination Conditions

CONTEXT_MANAGER terminates only after:
1. Phase 3 complete
2. All phase summaries generated and verified
3. Final token accounting complete
4. Context pruning metrics recorded
5. Cost savings documented
