# AffairsMap Orchestration System: New Agent Rulesets Index

**Date**: September 9, 2026  
**System**: 27-Agent Orchestrator (24 existing + 11 new)  
**Status**: Ready for implementation (Day 1-5 roadmap)

---

## Team F-2: Observability & Traceability (3 agents)

### 1. TRACER_RULESET.md
**Role**: Distributed tracing and execution observability  
**Start Event**: Phase 0 T+0:00  
**Key Responsibility**: Capture every agent action, tool call, state transition as structured spans (OpenTelemetry standard). Maintain write-once-read-many god log.

**Critical Sections**:
- Span Capture Protocol (5 span types)
- God Log Storage (SQLite append-only design)
- Distributed Tracing (OpenTelemetry compliance)
- Incident Replay Protocol (root cause analysis < 5 min)

**Success Metric**: 100% of agent actions traceable via unified god log; root cause analysis time < 5 minutes.

---

### 2. HANDOFF_AUDITOR_RULESET.md
**Role**: Context transfer validation and integrity verification  
**Start Event**: After every inter-agent handoff  
**Key Responsibility**: Verify that every agent-to-agent handoff transfers task context correctly. Ensure receiving agent has exact task scope, constraints, prior context.

**Critical Sections**:
- Handoff Protocol (3-stage: send → receipt → audit)
- Context Validation Criteria (4 dimensions: task scope, constraints, prior context, versions)
- Handoff Validation Matrix (per-phase validation rules)
- Packet Integrity Verification (hash-based validation)
- Recovery on Handoff Failure (error handling protocol)

**Success Metric**: 100% handoff accuracy; zero silent context loss events; 0% undetected corruption.

---

### 3. CONTEXT_MANAGER_RULESET.md
**Role**: Context window budgeting, pruning, and memory management  
**Start Event**: Phase 0 T+0:00  
**Key Responsibility**: Monitor context window utilization. Enforce explicit budgets per agent. Trigger summarization/pruning before limits to prevent cost explosion and hallucination.

**Critical Sections**:
- Per-Agent Allocation (9 core agents, ~650K total budget)
- Pruning Strategy (timeline-based summarization at phase boundaries)
- Summarization Rules (verbose → summary transformation)
- Pruning Triggers (automatic at 75% budget threshold)
- What to Prune vs What to Keep (retention matrix)
- Context Tracking Dashboard (real-time observability)

**Success Metric**: All agents operate within 75% of context window; zero context-overload hallucinations; 30%+ cost reduction vs unbounded.

---

## Team F-3: Resilience & Recovery (2 agents)

### 4. STATE_MANAGER_RULESET.md
**Role**: System state persistence and crash recovery  
**Start Event**: Phase 0 T+0:00  
**Key Responsibility**: Checkpoint system state at phase boundaries. Enable resume from last checkpoint if SIMON or any critical agent restarts. Eliminate 90% task failure risk.

**Critical Sections**:
- Checkpoint Strategy (4 checkpoints per run at T+0, T+90, T+150, T+210)
- What Gets Checkpointed (complete state snapshot)
- Storage Location (SQLite checkpoint table)
- Recovery Protocol (detect, load, verify, resume)
- No Duplicate Execution (idempotency guarantee)
- Checkpoint Verification (RECOVERY_AUDITOR validation)
- Partial Recovery (fallback strategy if checkpoint bad)

**Success Metric**: 100% checkpoint creation; zero data loss on recovery; 0% duplicate execution; recovery time < 5 minutes.

---

### 5. RECOVERY_AUDITOR_RULESET.md
**Role**: Checkpoint validation and recovery integrity verification  
**Start Event**: After each checkpoint creation  
**Key Responsibility**: Validate every checkpoint before it can be used for recovery. Verify no data loss, no corruption, no duplicate execution on resume.

**Critical Sections**:
- Validation Protocol (5 stages: hash, schema, referential integrity, logical consistency, completeness)
- Comprehensive Checklist (multi-point validation)
- Recovery Testing (before-recovery simulation)
- Error Handling During Recovery (recovery failure protocol)
- Checkpoint Comparison (diff analysis for debugging)

**Success Metric**: 100% checkpoint validation; zero undetected corruption; zero duplicate executions; recovery success rate 100%.

---

## Team F-4: Judge Council (3 agents)

### 6. TECH_JUDGE_RULESET.md
**Role**: Technical audit and build quality validation  
**Start Event**: Phase 3 complete, QA_GATE approval ready  
**Key Responsibility**: Independently audit technical readiness: build quality, performance metrics, security, API coverage. Vote GO/NO-GO on technical grounds.

**Critical Sections**:
- Audit Criteria (10 items: compilation, API coverage, schema, data integrity, performance, security, tests, logging, dependencies, docs)
- Voting Decision Matrix (criterion weighting, auto-fail conditions)
- Build Compilation (0 errors required)
- API Coverage (all 6 endpoints implemented)
- Database Schema Validation (matches spec exactly)
- Data Integrity (150 entities, 0 duplicates, 0 orphans)
- Performance Baselines (< 500ms p95 for digest, etc.)
- Security Baseline (parameterized queries, no hardcoded secrets, auth enforced)
- Test Coverage (>95% pass rate required)

**Success Metric**: Comprehensive technical audit; zero undetected build errors; performance validated; security baseline met.

---

### 7. PRODUCT_JUDGE_RULESET.md
**Role**: Product audit and user workflow validation  
**Start Event**: Phase 3 complete, QA_GATE approval ready  
**Key Responsibility**: Independently audit product readiness: features complete, UX working, user workflows intact, badge differentiation validated. Vote GO/NO-GO on product grounds.

**Critical Sections**:
- Audit Criteria (8 items: feature completeness, UX consistency, responsive design, loading states, test user feedback, content quality, pricing enforcement, differentiation)
- Voting Decision Matrix (criterion weighting, auto-fail conditions)
- Feature Completeness (6 required features: digest, entity graph, succession chains, scheme disambiguation, YoY index, badge tap-through)
- UX & Interaction (design tokens, interactive states)
- Responsive Design (mobile/tablet/desktop testing)
- Test User Validation (primary user workflow tested)
- Content Quality (first 7 days of digest verified)
- Pricing Enforcement (paid features locked for free users)
- Differentiation Validation (all 4 differentiators confirmed unique)

**Success Metric**: Comprehensive product audit; core user workflow validated; differentiators verified.

---

### 8. LAUNCH_JUDGE_RULESET.md
**Role**: Operational audit and launch safety validation  
**Start Event**: Phase 3 complete, QA_GATE approval ready  
**Key Responsibility**: Independently audit launch readiness: rollback exists, support infrastructure ready, SLA setup complete, monitoring configured. Vote GO/NO-GO on operations grounds.

**Critical Sections**:
- Audit Criteria (8 items: rollback procedure, monitoring, SLA definition, backup & DR, connection management, secrets, support runbook, communication)
- Voting Decision Matrix (criterion weighting, auto-fail conditions)
- Rollback Procedure (tested, < 15 min recovery time)
- Monitoring & Alerting (APM configured, custom metrics on badge tap)
- SLA Definition & Setup (99.5% availability, < 500ms digest latency, < 1% error rate)
- Backup & Disaster Recovery (hourly backups, tested restore, < 30 min RTO)
- Database Connection Management (pool configured, load-tested)
- Secrets Management (vault-stored, rotation scheduled, access logged)
- Support Runbook (procedures documented, team trained)
- Launch Communication Plan (announcement, feedback channel, support channel)

**Success Metric**: Comprehensive ops audit; rollback tested; monitoring active; SLA tracking enabled; zero operational surprises.

---

## Implementation Roadmap

### Day 1: Foundation
- [ ] Create TRACER_RULESET.md (god log + distributed tracing)
- [ ] Create STATE_MANAGER_RULESET.md (checkpoint/recovery protocol)
- [ ] Update ERROR_MONITOR to spawn RECOVERY_AUDITOR on failures

### Day 2: Validation
- [ ] Create HANDOFF_AUDITOR_RULESET.md
- [ ] Create CONTEXT_MANAGER_RULESET.md
- [ ] Update OPTIMIZER with predictive timeline logic

### Day 3: Judge Council
- [ ] Create TECH_JUDGE_RULESET.md
- [ ] Create PRODUCT_JUDGE_RULESET.md
- [ ] Create LAUNCH_JUDGE_RULESET.md
- [ ] Update QA_GATE to route decisions to judges

### Day 4: Testing
- [ ] Test handoff validation (Phase 0→1 transition)
- [ ] Test state recovery (simulate SIMON restart at T+120)
- [ ] Test judge voting (different vote outcomes)
- [ ] Test context pruning (verify cost reduction)

### Day 5: Launch
- [ ] Full 5.5h run with all 27 agents
- [ ] Verify traces complete and replay-able
- [ ] Verify checkpoints at T+90, T+150, T+210
- [ ] Verify judges vote unanimously GO

---

## Key Binding Principles

### New Principle #1: Observability First
Every agent action must be traceable. Build traces into spawn packets, not as afterthought.

### New Principle #2: State Persistence is Non-Negotiable
Long-running systems must checkpoint. Design for interruption as normal case.

### New Principle #3: Handoffs Require Explicit Validation
No agent starts work without HANDOFF_AUDITOR confirming context integrity.

### New Principle #4: Independent Judges Vote on Critical Decisions
QA_GATE is no longer final authority. Three independent judges must all vote GO before go-live.

### New Principle #5: Context Windows Are Budgets
Each agent has maximum context allocation. CONTEXT_MANAGER enforces pruning before limits hit.

---

## System Architecture (27 Agents Total)

### Core Teams (Existing)
- **SIMON** (1): Orchestrator
- **Dev Team** (5): DEV_EXECUTOR, DEV_DATA, DEV_UI, DEV_MONITOR, DEV_PERFORMANCE
- **QA Team** (3): QA_DATA, QA_VISUAL, QA_GATE
- **Curation Team** (2): CURATOR_TL, CURATOR_SOURCE
- **System Team** (3): RESOURCE_MONITOR, OPTIMIZER, ERROR_MONITOR

### New Teams (11 agents)
- **Observability & Traceability** (3): TRACER, HANDOFF_AUDITOR, CONTEXT_MANAGER
- **Resilience & Recovery** (2): STATE_MANAGER, RECOVERY_AUDITOR
- **Judge Council** (3): TECH_JUDGE, PRODUCT_JUDGE, LAUNCH_JUDGE
- **New Debuggers** (4): DEBUGGER_OBSERVABILITY, DEBUGGER_RESILIENCE, DEBUGGER_JUDGING, DEBUGGER_HANDOFF

### Updated Existing
- OPTIMIZER (now with predictive timeline management)
- ERROR_MONITOR (now triggers RECOVERY_AUDITOR on failures)
- QA_GATE (now routes to Judge Council, not directly to deployment)

---

## Success Criteria

### Observability (TRACER, HANDOFF_AUDITOR, CONTEXT_MANAGER)
- [ ] Every agent action traceable in god log
- [ ] Root cause analysis achievable in < 5 minutes
- [ ] No silent context loss events
- [ ] Context cost reduced 30%+ vs unbounded

### Resilience (STATE_MANAGER, RECOVERY_AUDITOR)
- [ ] All checkpoints validated before use
- [ ] Recovery success rate 100%
- [ ] Zero duplicate executions on recovery
- [ ] 90% improvement in task failure risk

### Validation (TECH_JUDGE, PRODUCT_JUDGE, LAUNCH_JUDGE)
- [ ] Independent validation on 3 dimensions (tech, product, ops)
- [ ] All judges vote GO before deployment
- [ ] Zero undetected failures at gate approval
- [ ] 7x accuracy improvement vs single-judge approval

---

## File Listing

```
/mnt/user-data/outputs/
├── TRACER_RULESET.md
├── HANDOFF_AUDITOR_RULESET.md
├── CONTEXT_MANAGER_RULESET.md
├── STATE_MANAGER_RULESET.md
├── RECOVERY_AUDITOR_RULESET.md
├── TECH_JUDGE_RULESET.md
├── PRODUCT_JUDGE_RULESET.md
├── LAUNCH_JUDGE_RULESET.md
└── AGENT_RULESETS_INDEX.md (this file)
```

All rulesets are standalone, independent documents. Each can be read in isolation or as part of the system. No cross-file dependencies.

---

## Integration Checkpoints

**After Day 1**: TRACER, STATE_MANAGER operational, can capture execution traces  
**After Day 2**: HANDOFF_AUDITOR, CONTEXT_MANAGER operational, can validate handoffs and manage budgets  
**After Day 3**: Judge Council operational, can vote GO/NO-GO before deployment  
**After Day 4**: All teams tested, integration verified  
**After Day 5**: Full system operational, ready for production use

---

## Next Steps for Udhay

1. **Review all 8 rulesets** (this document provides overview; detailed spec in each file)
2. **Confirm or adjust** agent responsibilities (e.g., which dimension is most critical for your launch?)
3. **Assign implementation** (which of your team will build each agent's logic?)
4. **Set timeline** (can you do Day 1-5 roadmap, or need extension?)
5. **Test checkpoint** (after Day 1, verify TRACER + STATE_MANAGER integration works)
6. **Full system run** (Day 5: run full 5.5h launch with all 27 agents)

---

**Questions? Each ruleset has a "Compliance Checklist" and "Integration Points" section for reference.**
