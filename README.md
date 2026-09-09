# Agent Rules

**27-Agent Orchestration System for AffairsMap Pre-Launch Phases**

Ruleset library for SIMON, the orchestrator that coordinates Phase 0–3 launch execution across 5 specialized teams. Defines behavior, constraints, success metrics, and failure modes for every agent in the system.

---

## Quick Start

### Invoke SIMON (Orchestrator)

```bash
SIMON, start Phase [X]. Focus: [task]. Expected: [metric]. Time: [estimate].
```

**Example**:
```bash
SIMON, start Phase 0. Focus: Ingest 150 entities, validate schema, backfill succession chains. Expected: 150 entities, 0 orphans, schema passes. Time: 90 min.
```

SIMON will:
1. Parse directive
2. Validate system state
3. Spawn agents with complete task briefs
4. Track execution every 30 min
5. Escalate blockers (max 5 min wait for Udhay decision)

### SIMON Initialization (Every Session)

Copy & paste into Claude:

```
You are SIMON, the AffairsMap orchestrator.

Before starting Phase work today, verify all systems are initialized.

Check EACH item below. If YES to all, respond:
✅ ALL SYSTEMS READY

If any NO, respond:
❌ SYSTEMS NOT READY

---

## INITIALIZATION CHECKLIST

### 1. Project Context
- [ ] /home/udhayakumar/projects/affairsmap-v6/ exists with AGENTS.md, affairsmap-pipeline/, web/, agent-rules/

### 2. AGENTS.md Current
- [ ] AGENTS.md read TODAY
- [ ] Agent Rules section exists
- [ ] Today's Phase focus confirmed

### 3. Git Status
- [ ] affairsmap-pipeline/ git status clean
- [ ] web/ git status clean
- [ ] Parent is NOT a git repo

### 4. Data Pipeline Ready
- [ ] Python environment active
- [ ] compile.py ready
- [ ] SQLite database exists

### 5. Frontend Ready
- [ ] Next.js dev server ready or built
- [ ] Routes: /, /[type], /[type]/[slug], /search, /compare
- [ ] Entity types match pipeline TYPE_REGISTRY

### 6. Agent Rules Available
- [ ] TRACER accessible
- [ ] STATE_MANAGER accessible
- [ ] TECH/PRODUCT/LAUNCH judges available

### 7. Today's Focus Confirmed
- [ ] Phase number (0, 1, 2, or 3)
- [ ] Task stated
- [ ] Success metric defined
- [ ] Time estimated
```

---

## System Architecture

### Team Breakdown (27 Agents)

```
┌─────────────────────────────────────────────────────────────────────┐
│                          SIMON (Orchestrator)                        │
│                    Pure orchestration, no execution work              │
│                                                                       │
│  Responsibilities: Parse directives, spawn agents, track state,      │
│                  resolve conflicts, escalate blockers                │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
                    ▼               ▼               ▼
        ┌──────────────────┐ ┌──────────────┐ ┌──────────────────┐
        │   Core Teams     │ │  Observability│ │  Resilience &    │
        │   (14 agents)    │ │  & Traceability
        │                  │ │  (3 agents)  │ │  Recovery        │
        │                  │ │              │ │  (2 agents)      │
        └──────────────────┘ └──────────────┘ └──────────────────┘
                    │               │               │
                    ▼               ▼               ▼
        ┌──────────────────────────────────────────────────────────┐
        │                    JUDGE COUNCIL                          │
        │           (3 independent vote: GO/NO-GO)                 │
        │                                                           │
        │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
        │  │ TECH_JUDGE  │  │PRODUCT_JUDGE│  │ LAUNCH_JUDGE│     │
        │  │  Audits:    │  │  Audits:    │  │  Audits:    │     │
        │  │ • Build     │  │ • Features  │  │ • Rollback  │     │
        │  │ • APIs      │  │ • UX        │  │ • Monitoring│     │
        │  │ • Schema    │  │ • Workflows │  │ • SLA       │     │
        │  │ • Data      │  │ • Diff.     │  │ • Backup    │     │
        │  │ • Perf      │  │ • Pricing   │  │ • Secrets   │     │
        │  │ • Security  │  │             │  │ • Runbook   │     │
        │  │ • Tests     │  │             │  │             │     │
        │  └─────────────┘  └─────────────┘  └─────────────┘     │
        └──────────────────────────────────────────────────────────┘
```

### Core Teams (14 Agents)

**Team A: Dev (5 agents)**
- DEV_EXECUTOR: Phase lead, orchestrates team tasks
- DEV_DATA: Data pipeline, entity ingestion, schema validation
- DEV_UI: Routes, components, responsive design
- DEV_MONITOR: Monitoring, logging, observability
- DEV_PERFORMANCE: Metrics, optimization, load testing

**Team B: QA (3 agents)**
- QA_DATA: Entity validation, data integrity, test coverage
- QA_VISUAL: UI validation, responsive design, accessibility
- QA_GATE: Phase approval, blocks invalid progress to next phase

**Team C: Curation (2 agents)**
- CURATOR_TL: Source validation, conflict resolution
- CURATOR_SOURCE: Source authenticity, fact-checking

**Team D: System (3 agents)**
- RESOURCE_MONITOR: Capacity, parallel work load, agent health
- OPTIMIZER: Timeline prediction, phase compression recommendations
- ERROR_MONITOR: Failure detection, spawns DEBUGGER agents

**Continuous Agent**:
- AUDITOR: Continuous scan (bugs, tasks, SEO gaps, perf issues)

### New Teams (5 Agents)

**Team E: Observability & Traceability (3 agents)**
- TRACER: Distributed tracing, god log, span capture (OpenTelemetry)
- HANDOFF_AUDITOR: Context transfer validation, 3-stage protocol
- CONTEXT_MANAGER: Context budget enforcement, auto-pruning at 75%

**Team F: Resilience & Recovery (2 agents)**
- STATE_MANAGER: Checkpoint creation (4 checkpoints per run)
- RECOVERY_AUDITOR: Checkpoint validation (5-stage protocol)

**Debug Team (8 agents, on-call)**
- DEBUGGER_SYSTEM, DEBUGGER_OBSERVABILITY, DEBUGGER_RESILIENCE, DEBUGGER_JUDGING, DEBUGGER_HANDOFF, + 3 more
- Spawned by ERROR_MONITOR when agent blocks

---

## Phase Timeline

```
T+0:00 ┌─────────────────────────────────────────────────────────┐
       │ Session Start: AUDITOR spawns, audit-report.md created   │
       └─────────────────────────────────────────────────────────┘
       │
       ▼
T+0:00 ┌──────────────────────────────────────────────────────────┐
       │ PHASE 0: Data Pipeline (90 min)                           │
       │ • Ingest 150 entities                                     │
       │ • Backfill succession chains                              │
       │ • Validate schema                                         │
       │ Output: 150 entities + validation report                  │
       │ Teams: DEV_DATA, DEV_MONITOR, TRACER, STATE_MANAGER       │
       └──────────────────────────────────────────────────────────┘
       │
T+90:00├──────────────────────────────────────────────────────────┐
       │ CP1 Checkpoint: Phase 0 complete + validated outputs     │
       ├──────────────────────────────────────────────────────────┤
       │ PHASE 1: UI & Routes (60 min)                             │
       │ • Build routes + components                               │
       │ • Data to frontend                                        │
       │ • Test coverage                                           │
       │ Output: Routes, components, test suite                    │
       │ Teams: DEV_UI, QA_DATA, QA_VISUAL                         │
       └──────────────────────────────────────────────────────────┘
       │
T+150:00├──────────────────────────────────────────────────────────┐
        │ CP2 Checkpoint: Phase 1 complete + UI artifacts         │
        ├──────────────────────────────────────────────────────────┤
        │ PHASE 2: Performance & SEO (60 min)                      │
        │ • Responsive design                                      │
        │ • Load testing                                           │
        │ • SEO schema (JSON-LD)                                   │
        │ • Rollback procedure                                     │
        │ Output: Metrics, SEO schema, rollback plan               │
        │ Teams: DEV_PERFORMANCE, QA_VISUAL, CURATOR_TL            │
        └──────────────────────────────────────────────────────────┘
        │
T+210:00├──────────────────────────────────────────────────────────┐
        │ CP3 Checkpoint: Phase 2 complete + deployment ready     │
        ├──────────────────────────────────────────────────────────┤
        │ PHASE 3: Final Validation (45 min)                       │
        │ • QA_GATE comprehensive check                            │
        │ • Judge Council votes                                    │
        │ • Deployment approval                                    │
        │ Output: GO/NO-GO decision                                │
        │ Teams: QA_GATE, TECH_JUDGE, PRODUCT_JUDGE, LAUNCH_JUDGE │
        └──────────────────────────────────────────────────────────┘
        │
T+255:00└──────────────────────────────────────────────────────────┐
         │ Deployment: DEV_EXECUTOR deploys to production         │
         └──────────────────────────────────────────────────────────┘

TOTAL: ~5 hours (Phase 0-3 + deployment overhead)
```

---

## Agent Spawning & Dependencies

```
                          SIMON (Orchestrator)
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
                 Spawn         Track          Escalate
               Agents          Status          Blockers
                    │             │             │
        ┌───────────┴────┬────────┴────┬───────┴────┐
        │                │             │            │
        ▼                ▼             ▼            ▼
    AUDITOR        Phase 0 Teams   Monitor Log    Udhay
  (Continuous)   (Parallel work)   (Every 30min) (5min max)
        │                │             │            │
        │                │             │            ▼
        │                ├────┬────┬───┤      Decision
        │                │    │    │   │         │
        │                ▼    ▼    ▼   ▼         │
        │           Dev  QA  Curation System      │
        │           Teams Teams Teams Teams       │
        │                │    │    │   │          │
        │                └────┴────┴───┘          │
        │                     │                   │
        │    ┌────────────────┼──────────────┐    │
        │    │                │              │    │
        ▼    ▼                ▼              ▼    ▼
    audit- Phase 1        Phase 2         Phase 3
    report Teams          Teams           Teams
     (Top 5) │              │               │
            ▼              ▼               ▼
        TRACER ──────────────────────────→ JUDGE COUNCIL
        (God Log)                    (All votes must be GO)
             │
             ├─→ HANDOFF_AUDITOR (Validate handoffs)
             ├─→ CONTEXT_MANAGER (Budget enforcement)
             ├─→ STATE_MANAGER (Checkpoint creation)
             └─→ RECOVERY_AUDITOR (Checkpoint validation)
```

---

## Escalation Protocol

```
┌────────────────────────────────────────────────────────────────┐
│                   SIMON Detects Blocker                         │
└────────────────────────────────────────────────────────────────┘
                            │
                    ┌───────▼────────┐
                    │ What is it?    │
                    └───────┬────────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
            ▼               ▼               ▼
    ┌──────────────┐ ┌──────────────┐ ┌─────────────────┐
    │ Critical     │ │ Priority     │ │ Resource        │
    │ Blocker      │ │ Conflict     │ │ Contention      │
    │              │ │ (competing   │ │ (<20% capacity) │
    │ (Phase       │ │  teams)      │ │                 │
    │  blocked)    │ │              │ │                 │
    └──────┬───────┘ └──────┬───────┘ └────────┬────────┘
           │                │                 │
           ▼                ▼                 ▼
    ┌──────────────────────────────────────────────────┐
    │ T+0:00 → Escalate to Udhay                       │
    │ With 2-3 options: "A) [option], B) [option]"    │
    │ Recommend: [A/B]                                 │
    └──────────────────────────────────────────────────┘
           │
           ├─────────────────────────────────────────┐
           │                                         │
    ┌──────▼──────┐                           ┌─────▼───────────┐
    │ T+0:00-5:00 │                           │ T+5:00+ No Resp. │
    │   WAIT      │                           │ EXECUTE DEFAULT  │
    │ for Udhay   │                           │ ACTION AUTO      │
    │ decision    │                           │                  │
    └──────┬──────┘                           └────────┬──────────┘
           │                                           │
           ▼                                           ▼
    ┌──────────────────────────────────────────────────┐
    │ Udhay responds                                   │
    │ Option A → SIMON executes A                      │
    │ Option B → SIMON executes B                      │
    │ Later override → SIMON updates retroactively    │
    └──────────────────────────────────────────────────┘
```

---

## Status Report (Every 30 Minutes)

SIMON reports in **≤5 lines**:

```
T+45:00 Status: Phase 0 = 50/90 min. Entities: 145/150 (97%). 
        Blocker: 5 orphaned position refs (CRITICAL). 
        Capacity: 85%. Next: QA data validation @ T+75.
```

**What it includes**:
- Time elapsed
- Active phases + progress
- Blockers (if any)
- Resource capacity
- Next milestone

---

## Checkpoint & Recovery

### Checkpoint Strategy (4 per run)

```
CP0 (T+0)          CP1 (T+90)        CP2 (T+150)       CP3 (T+210)
└────────┐          └───────┐        └───────┐         └───────┐
         │                  │                │                 │
    Initial        Phase 0           Phase 1           Phase 2
    spawn          complete          complete          complete
    config         + validation       + UI              + deployment
                   + entities         artifacts         ready
                                                        
Size: 1 MB         Size: 5 MB         Size: 8 MB        Size: 6 MB
                                      
Hash Verified ✓    Hash Verified ✓    Hash Verified ✓   Hash Verified ✓
(RECOVERY_AUDITOR)
```

### Recovery Flow (On Restart)

```
SIMON Restarts
     │
     ▼
Query agent_executions table
     │
     ├─────────────────────────────────────────┐
     │                                         │
     ▼                                         ▼
  Find idempotency_key                   Key not found?
  for each task                               │
     │                                        ▼
     ├─ Status = 'complete'              Spawn fresh agent
     │  → Skip (already done)              (start from T+0)
     │
     ├─ Status = 'in_progress'
     │  → Resume from last checkpoint
     │     (STATE_MANAGER provides state)
     │
     └─ Key not found?
        → Spawn fresh agent

Result: Zero duplicate work. Full recovery from checkpoint in < 5 min.
```

---

## Judge Council Voting (Phase 3)

```
After Phase 2 complete → Three independent judges audit

┌──────────────────────────────────────────────────────────────┐
│                    TECH_JUDGE Audits                         │
│ Build (0 errors) • APIs (all 6 impl.) • Schema (matches)     │
│ Data (150 ent., 0 dup.) • Perf (p95 < 500ms)                │
│ Security (no hardcoded secrets) • Tests (>95% pass)          │
│                                                              │
│ Vote: GO ✓ / NO-GO ✗                                        │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                    PRODUCT_JUDGE Audits                      │
│ Features (digest, graph, chains, disambiguation, YoY, badge)│
│ UX (design tokens, states) • Responsive (mobile/tablet/desk)│
│ Workflows (primary user path tested) • Content (7 days ✓)    │
│ Differentiation (all 4 verified unique)                      │
│                                                              │
│ Vote: GO ✓ / NO-GO ✗                                        │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                    LAUNCH_JUDGE Audits                       │
│ Rollback (tested, < 15 min RTO) • Monitoring (APM active)    │
│ SLA (99.5%, < 500ms, < 1% error) • Backup (hourly, RTO 30m) │
│ Secrets (vault-stored) • Runbook (documented, team trained)  │
│                                                              │
│ Vote: GO ✓ / NO-GO ✗                                        │
└──────────────────────────────────────────────────────────────┘
              │                │                │
              ▼                ▼                ▼
          Result           Result           Result
            │                │                │
            └────────────────┼────────────────┘
                             │
                    ┌────────▼────────┐
                    │ All GO?         │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │                 │
                  YES               NO
                    │                 │
                    ▼                 ▼
              ✓ Deploy          ✗ Escalate to Udhay
           (DEV_EXECUTOR)      (Ship with gap or extend?)
```

---

## Continuous Scanning (AUDITOR)

```
Every Session Start (T+0:00)
            │
            ▼
    AUDITOR Spawns
            │
    ┌───────┼───────┐
    │       │       │
Every 10min │ Every 30min │ Every 60min
    │       │       │
    ▼       ▼       ▼
Build      TODOs    Perf
errors     FIXMEs   regressions
TypeScript Orphaned SEO gaps
errors     entities Accessibility
Missing    Empty    Test
files      data     coverage
404 routes files    Dependency
           Route    updates
           failures
           Component
           render
           failures
            │
            └────────┬────────────┐
                     │            │
                     ▼            ▼
            audit-report.md  Continuous
            updated every    updates
            30 min           during phases
                     │
                     ▼
            Top 5 Most Urgent
            listed first
                     │
            ┌────────┴────────┐
            │                 │
        Fixed → ✅ FIXED    CRITICAL
        marked  (T+[time])    │
                              ▼
                        Escalate to Udhay
                        immediately (<5 min)
```

---

## Context Budgeting (CONTEXT_MANAGER)

```
Total Budget: 650 KB per session

Allocation:
  SIMON (orchestrator):              50 KB
  TRACER (god log):                 100 KB
  Dev Agents (4 × 50KB):            200 KB
  QA Agents (3 × 33KB):             100 KB
  Curators (2 × 37KB):               75 KB
  Judges (3 × 25KB):                 75 KB
  Debuggers (8 × 12KB):             100 KB
                                    ────────
                                    700 KB
                                    (5% buffer)

Enforcement:
  < 75% usage → Proceed normally
  75% usage   → Summarize + prune oldest logs
  At limit    → Pause agent, compact context, resume

Result: Zero hallucination from context overload. 30%+ cost reduction.
```

---

## File Reference

| File | Purpose | Read Time |
|------|---------|-----------|
| **_INDEX.md** | System overview, team breakdown, roadmap | 10 min |
| **simon.md** | SIMON mandate, spawning, escalation, state | 15 min |
| **AUDITOR_PROTOCOL.md** | AUDITOR binding rule (mandatory spawn) | 3 min |
| **auditor-agent.md** | AUDITOR detailed specification | 5 min |
| **tracer.md** | Distributed tracing, god log, spans | 8 min |
| **handoff_auditor.md** | Context validation protocol | 9 min |
| **context_manager.md** | Budget allocation, pruning strategy | 9 min |
| **state_manager.md** | Checkpoints, recovery protocol | 9 min |
| **recovery_auditor.md** | Checkpoint validation, recovery testing | 9 min |
| **tech_judge.md** | Technical audit criteria, voting | 10 min |
| **product_judge.md** | Product audit criteria, voting | 10 min |
| **launch_judge.md** | Operational audit criteria, voting | 11 min |

**Recommended reading**:
- **New to system?** Read _INDEX.md + simon.md (25 min)
- **Need specific agent?** Jump to ruleset file
- **Quick start?** Use simon-autoload.txt

---

## Standing Rules (Never Relax)

1. **AUDITOR mandatory** — Every session spawn at T+0:00
2. **SIMON pure orchestration** — Never executes work directly
3. **Ruleset with every spawn** — Agent doesn't start without `ruleset_loaded: true`
4. **Context budgets enforced** — No agent exceeds allocation
5. **Checkpoints validated** — RECOVERY_AUDITOR validates before use
6. **Idempotency prevents duplicates** — Same task spawned twice? Key prevents re-execution
7. **Escalations wait 5 min** — Then execute default action
8. **Phase N+1 waits for Phase N** — QA_GATE approval before spawning next
9. **Judges vote independently** — All three must vote GO
10. **Timeline is fixed** — 5 hours total; escalate if overrunning

---

## Getting Help

**Find specific agent rules?** Check _INDEX.md "System Architecture" section for team breakdown + file mapping.

**Agent blocked mid-phase?** ERROR_MONITOR spawns DEBUGGER_[ROLE] to investigate.

**Need to update ruleset?** Read full ruleset first, make surgical edits, update "Integration Points" section for dependent agents.

**Judge voted NO-GO?** Read judge's audit criteria section (tech_judge.md, product_judge.md, or launch_judge.md) to see why.

---

## Attribution & Changes

**Last updated**: 2026-09-09  
**System status**: Ready for Phase 0 orchestration run  
**Documentation**: Complete (8 rulesets + CLAUDE.md + README.md)

To contribute changes:
1. Identify affected ruleset(s)
2. Read complete ruleset (context matters)
3. Make surgical edits only
4. Update "Integration Points" if other agents affected
5. Commit: `docs: update [ruleset] (specific change)`

---

## Quick Links

- **_INDEX.md** — Start here for system overview
- **simon.md** — SIMON orchestration rules
- **simon-autoload.txt** — Copy-paste SIMON initialization checklist
- **CLAUDE.md** — Internal Claude Code guidance (for AI agents operating here)

---

**Status**: ✅ System ready. Brief SIMON to start Phase work.
