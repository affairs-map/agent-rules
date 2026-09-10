# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

---

## Project Overview

**agent-rules/** is the ruleset library for the **SIMON Orchestrator**, a 27-agent system that coordinates AffairsMap's pre-launch phases (Phases 0–3 + deployment gate).

**What this directory contains**:
- Agent behavior rulesets (markdown specifications)
- SIMON initialization checklists
- System architecture documentation
- No code, no build artifacts—pure documentation

**Key role**: Each agent in SIMON operates under a binding ruleset defined in this directory. The rulesets specify what each agent must do, what constraints apply, success metrics, and failure modes.

---

## Architecture Overview

### SIMON: The Orchestrator (27-Agent System)

**Core Concept**: SIMON is pure orchestration. It parses incoming directives, spawns agents with complete task briefs, tracks execution, and escalates blockers to Udhay. All execution work is done by sub-agents in parallel.

**Five teams**:

1. **Core Teams (existing, 14 agents)**:
   - SIMON (orchestrator)
   - Dev Team (5): DEV_EXECUTOR, DEV_DATA, DEV_UI, DEV_MONITOR, DEV_PERFORMANCE
   - QA Team (3): QA_DATA, QA_VISUAL, QA_GATE
   - Curation Team (2): CURATOR_TL, CURATOR_SOURCE
   - System Team (3): RESOURCE_MONITOR, OPTIMIZER, ERROR_MONITOR

2. **New Observability & Traceability Team (3 agents)**:
   - **TRACER**: Captures every agent action as structured spans (OpenTelemetry standard); maintains write-once god log
   - **HANDOFF_AUDITOR**: Validates context transfer integrity between agents (3-stage: send → receipt → audit)
   - **CONTEXT_MANAGER**: Monitors context budgets per agent; auto-triggers pruning at 75% threshold

3. **New Resilience & Recovery Team (2 agents)**:
   - **STATE_MANAGER**: Checkpoints system state at phase boundaries (4 checkpoints per run); enables recovery from interruption
   - **RECOVERY_AUDITOR**: Validates every checkpoint before it can be used (5-stage validation protocol)

4. **New Judge Council (3 agents)**:
   - **TECH_JUDGE**: Audits technical readiness (build, API coverage, schema, data, performance, security, tests)
   - **PRODUCT_JUDGE**: Audits product readiness (features, UX, responsive design, user workflows, differentiation)
   - **LAUNCH_JUDGE**: Audits operational readiness (rollback, monitoring, SLA, backup, secrets, runbook)

5. **Debuggers (8 agents)**:
   - DEBUGGER_SYSTEM, DEBUGGER_OBSERVABILITY, DEBUGGER_RESILIENCE, DEBUGGER_JUDGING, DEBUGGER_HANDOFF, + 3 more
   - Spawned by ERROR_MONITOR when an agent fails or blocks; investigate and suggest recovery

### Phase Timeline

| Phase | Timing | Teams Active | Output |
|-------|--------|-------------|--------|
| **Session Start** | T+0:00 | AUDITOR (continuous scan) | audit-report.md |
| **Phase 0** | T+0:00 → T+90:00 | Dev (data pipeline) | 150 entities + success chains |
| **Phase 1** | T+90:00 → T+150:00 | Dev (UI layer) + QA_DATA | Routes, components, tests |
| **Phase 2** | T+150:00 → T+210:00 | Dev (performance polish) + QA_VISUAL | Responsive design, SEO, metrics |
| **Phase 3** | T+210:00 → T+255:00 | QA_GATE (final validation) | Gate approval or rejection |
| **Deployment** | T+255:00+ | DEV_EXECUTOR (if GO) | Live on production |

**Hard timeline: 5 hours total.** No phase exceeds 1.5 hours. If overrunning, SIMON escalates to Udhay.

---

## File Structure

```
agent-rules/
├── _INDEX.md                          # Main index (start here)
├── AGENTS.md                          # This file
├── simon.md                           # SIMON orchestrator ruleset
├── AUDITOR_PROTOCOL.md                # AUDITOR binding rule (mandatory spawn)
├── auditor-agent.md                   # AUDITOR detailed spec
├── tracer.md                          # TRACER distributed tracing ruleset
├── handoff_auditor.md                 # HANDOFF_AUDITOR context validation ruleset
├── context_manager.md                 # CONTEXT_MANAGER budget enforcement ruleset
├── state_manager.md                   # STATE_MANAGER checkpoint/recovery ruleset
├── recovery_auditor.md                # RECOVERY_AUDITOR checkpoint validation ruleset
├── tech_judge.md                      # TECH_JUDGE technical audit ruleset
├── product_judge.md                   # PRODUCT_JUDGE product audit ruleset
├── launch_judge.md                    # LAUNCH_JUDGE operational audit ruleset
├── simon-autoload.txt                 # Quick SIMON initialization checklist
└── ui.md                              # UI rules and design tokens
```

**Reading order**:
1. **New to SIMON?** Start with `_INDEX.md` (10 min read, comprehensive overview)
2. **Need a specific agent?** Find its ruleset in the list above
3. **Starting a phase?** Read `simon.md` (spawning protocol, escalation matrix)
4. **Need quick checklist?** Use `simon-autoload.txt` (initialization template)

---

## Key Binding Principles

### Principle 1: Observability First
Every agent action must be traceable. Traces built into spawn packets, not afterthought. TRACER maintains god log; root cause analysis < 5 min.

### Principle 2: State Persistence is Non-Negotiable
Long-running systems must checkpoint. Design for interruption as normal case. STATE_MANAGER creates checkpoints at phase boundaries; RECOVERY_AUDITOR validates before use.

### Principle 3: Handoffs Require Explicit Validation
No agent starts work without HANDOFF_AUDITOR confirming context integrity. 3-stage protocol: send → receipt → audit.

### Principle 4: Independent Judges Vote on Critical Decisions
QA_GATE no longer final authority. Three independent judges (TECH_JUDGE, PRODUCT_JUDGE, LAUNCH_JUDGE) must ALL vote GO before go-live.

### Principle 5: Context Windows Are Budgets
Each agent has max context allocation (~650K total budget across 27 agents). CONTEXT_MANAGER enforces pruning before limits hit; prevents hallucination from context overload.

### Principle 6: SIMON is Pure Orchestration Only
**BINDING CONSTRAINT**: SIMON never executes work itself. SIMON's only role: spawn agents, track status, route escalations. If SIMON finds itself executing, it has made an error—correct by spawning appropriate agent.

### Principle 7: No Agent Grades Its Own Work — Four-Eyes Verification Required
**BINDING CONSTRAINT** (established governance practice: four-eyes/maker-checker principle, ISO/IEC/IEEE 29119-3):

No agent can report completion or validation of another agent's output without meeting these three conditions:
1. **Separate agent instance**: The verifying agent must be a structurally distinct spawn (different agent ID), not the same agent re-reading its own report.
2. **Evidence artifacts, not prose**: Verification claims must be backed by named artifacts (diff image, snapshot file, log file, screenshot) linked in the output, not just prose summary ("looks good").
3. **Independent re-derivation**: The verifying agent re-derives the list of what to verify (e.g., route list from compiled data, not copied from builder's list) and re-runs its own captures/checks rather than accepting the builder's artifacts as proof.

**Application**:
- DEV_UI builds and reports → QA_VISUAL independently verifies and re-checks (separate agent, artifacts required, independent route enumeration)
- DEV_DATA outputs entities → QA_DATA independently validates schema and duplicates (separate agent, artifact logs required)
- Any agent self-grading its own work (builder reporting "I verified myself") → automatic FAIL, cannot proceed

**Citation**: This principle adopts the four-eyes verification standard from audit/security governance and the artifact-evidence requirement from ISO/IEC/IEEE 29119-3 (test documentation standard).

---

## Common Workflows

### Starting a Phase

**How to invoke SIMON**:
```
SIMON, start Phase [X]. Focus: [task]. Expected: [metric]. Time: [estimate].
```

**SIMON's response**:
1. Parse directive
2. Validate system state (check dependencies, prior phase completion)
3. Spawn agents with complete task briefs (including ruleset file + success criteria)
4. Track execution every 30 min
5. Escalate to Udhay if blocker detected (wait max 5 min for decision)

**Example**:
```
SIMON, start Phase 0. Focus: Ingest 150 entities from data pipeline, validate schema, backfill succession chains. Expected: 150 entities all validated, 0 orphans, schema passes. Time: 90 min.
```

### Escalation Workflow

When SIMON encounters a blocker:
1. **T+0:00** → SIMON escalates with 2–3 options: "A) [option], B) [option]. Recommend: [A/B]"
2. **T+0:00 to T+5:00** → Wait for Udhay decision
3. **T+5:00+** → No response? Execute recommended default action automatically
4. **Later** → If Udhay responds: Override retroactively

**Escalation conditions** (escalate without waiting for agent feedback):
- Two failed debug attempts on same blocker
- Conflicting priorities (Phase 0 blocker found during Phase 1)
- Resource contention (<20% capacity)
- QA_GATE rejects output
- External blockers (Vercel outage, etc.)

### Status Checks

SIMON reports status every 30 min in **≤5 lines**:
- Time elapsed
- Active phases + progress
- Blockers
- Resource capacity
- Next milestone

**Example**:
```
Phase 0: 60/90 min. Entities: 145/150 (97%). Blocker: 5 orphaned position refs. Capacity: 85%. Next: QA data validation @ T+75.
```

### Context Pruning (Auto-Triggered)

CONTEXT_MANAGER monitors per-agent allocation:
- **< 75% used**: Agent proceeds normally
- **75% used**: Summarize oldest logs, prune to 50%
- **At budget limit**: Pause agent, compact context, resume

**Success metric**: All agents operate within 75% of budget; zero context-overload hallucinations; 30%+ cost reduction.

### Checkpoint & Recovery

STATE_MANAGER creates 4 checkpoints:
- **CP0 (T+0)**: Phase 0 spawn config
- **CP1 (T+90)**: Phase 0 complete + 150 entities
- **CP2 (T+150)**: Phase 1 complete + UI artifacts
- **CP3 (T+210)**: Phase 2 complete + deployment ready

If SIMON crashes mid-run:
1. On restart, query `agent_executions` table
2. Resume from last checkpoint (verified by RECOVERY_AUDITOR)
3. Skip agents already complete (idempotency key prevents re-execution)
4. **Zero duplicate work guaranteed**

### Judge Council Voting

After Phase 3 complete, three judges independently audit:

| Judge | Audits | Vote Criteria |
|-------|--------|---------------|
| TECH_JUDGE | Build (0 errors), APIs (all 6 implemented), schema (matches spec), data (150 entities, 0 duplicates), perf (p95 < 500ms), security (no hardcoded secrets), tests (>95% pass) | GO if all pass; NO-GO if any fail |
| PRODUCT_JUDGE | Features (digest, graph, chains, disambiguation, YoY, badges), UX (design tokens, states), responsive (mobile/tablet/desktop), workflows (primary user path), content (7 days verified), pricing (paid features locked for free) | GO if all pass; NO-GO if differentiation weak |
| LAUNCH_JUDGE | Rollback (tested, < 15 min RTO), monitoring (APM + custom metrics), SLA (99.5%, < 500ms, < 1% error), backup (hourly, tested restore < 30 min RTO), secrets (vault-stored), runbook (documented, team trained) | GO if all pass; NO-GO if gaps in runbook |

**Deployment rule**: All three judges must vote GO. If even one votes NO-GO, escalate to Udhay with specific feedback.

---

## Editing & Updating Rulesets

### When to Update a Ruleset

- Agent behavior changes (e.g., new escalation condition)
- New success metric added by Udhay
- Failure mode discovered and addressed
- Integration point changes (e.g., TRACER now includes new span type)

### How to Update

1. **Identify affected ruleset** (e.g., TRACER if tracing changes)
2. **Read entire ruleset** (understand context, don't edit in isolation)
3. **Make surgical edits** (change only the section affected)
4. **Update "Success Criteria" section** if metric changes
5. **Update "Integration Points" section** if other agents affected
6. **Commit with message**: `docs: update TRACER ruleset (add new span type)`

### Cross-Ruleset Dependencies

Some rulesets depend on others:
- HANDOFF_AUDITOR depends on TRACER (uses god log for validation)
- STATE_MANAGER depends on TRACER (traces every checkpoint)
- RECOVERY_AUDITOR depends on STATE_MANAGER (validates checkpoints)
- All agents depend on CONTEXT_MANAGER (budgets enforced globally)

**When editing a core ruleset**: Check `_INDEX.md` "Integration Checkpoints" section to find all dependent rulesets. Update them if needed.

---

## Standing Rules

1. **AUDITOR is mandatory**: Every session spawn AUDITOR at T+0:00. No exceptions. It continuously scans for bugs, tasks, opportunities.

2. **SIMON only spawns**: SIMON never executes work directly. If it finds itself executing, spawn appropriate agent instead.

3. **Every spawn packet includes ruleset**: Agent does not start work until it confirms receipt of complete ruleset + `ruleset_loaded: true`.

4. **No agent exceeds context budget**: CONTEXT_MANAGER enforces hard limits. Violations trigger auto-pruning.

5. **Checkpoints must be validated**: RECOVERY_AUDITOR validates every checkpoint before it can be used for recovery. No checkpoint used unvalidated.

6. **Idempotency prevents duplicate work**: Every agent spawn has unique idempotency key. On recovery, query by key to avoid re-execution.

7. **Escalations wait max 5 min**: If Udhay doesn't respond in 5 min, SIMON executes default action. Udhay can override later.

8. **Phase N+1 waits for Phase N output**: SIMON checks Phase N QA_GATE approval before spawning Phase N+1.

9. **Judges vote GO/NO-GO independently**: No judge sees other judges' votes until all three submitted. Prevents anchoring bias.

10. **Timeline is fixed**: 5 hours total, no phase > 1.5 hours. If overrunning, escalate immediately.

11. **No agent grades its own work — four-eyes required** (Principle 7): Verifying agent must be separate, report must include artifacts (not prose), and verifier re-derives and re-checks independently. Self-grading = automatic FAIL.

---

## Troubleshooting

### Agent Spawned But No Progress After 10 Min

1. Check agent status via TRACER (query god log for agent's last action)
2. If no recent span: Spawn DEBUGGER_[ROLE] to investigate
3. Common causes: Missing ruleset, ambiguous task brief, resource contention
4. SIMON escalates to Udhay if debug attempts fail twice

### Checkpoint Validation Fails (RECOVERY_AUDITOR rejects)

1. RECOVERY_AUDITOR performs 5-stage validation: hash, schema, referential integrity, logical consistency, completeness
2. Check AUDITOR report for what failed
3. Common causes: Partial write during checkpoint, memory corruption, data dependency violation
4. Recovery fallback: SIMON skips rejected checkpoint, resumes from prior good checkpoint (CP0 if all fail)

### Phase Running Over Timeline

1. OPTIMIZER predicts drift every 30 min
2. If phase is >15 min behind at 60% completion: Auto-recommend compression
3. **Compression options**: Reduce sub-tasks, parallelize more, extend next phase
4. 45 min behind → Escalate to Udhay; 60 min behind → Critical escalation

### Context Budget Hit During Phase

1. CONTEXT_MANAGER auto-pauses agent
2. Summarizes oldest logs (verbose → summary)
3. Prunes to 50% of budget
4. Resumes agent
5. Report: "Agent [X] hit budget at T+[time]. Compacted context. Resuming."

### AUDITOR Finding New Critical Issue Mid-Phase

1. AUDITOR updates audit-report.md with new issue
2. Marks severity (CRITICAL, HIGH, MEDIUM, LOW)
3. SIMON checks report at next 30-min status point
4. If CRITICAL: SIMON escalates to Udhay immediately (don't wait for next checkpoint)

### Judge Votes Split (Not Unanimous GO)

1. If TECH_JUDGE votes NO: Specific failure in build, APIs, schema, data, perf, security, or tests. Fix identified; QA_GATE cannot approve until fixed.
2. If PRODUCT_JUDGE votes NO: Feature incomplete or differentiation weak. Udhay decides: ship with gap or extend Phase 3.
3. If LAUNCH_JUDGE votes NO: Ops readiness issue (rollback untested, monitoring missing, etc.). Udhay decides: ship with ops gap or extend Phase 3 for ops prep.

All three must vote GO before deployment. No exceptions.

---

## Execution Model

### Sequential vs. Parallel

- **Sequential**: Phases 0 → 1 → 2 → 3 (phase gates ensure readiness)
- **Within phase**: Teams work in parallel (Dev, QA, Curation do parallel work within same phase)
- **Agents within team**: Can parallelize (e.g., DEV_UI + DEV_PERFORMANCE both in Phase 2)

**RESOURCE_MONITOR ensures**: No more than 20 agents active simultaneously (prevents context explosion).

### Idempotency & Recovery

Every agent spawn has:
- Unique idempotency key: `{phase}_{agent_id}_{task_id}_{timestamp}`
- Logged in `agent_executions` table immediately
- On recovery: Query by key → if status='complete', skip; if status='in_progress', resume from checkpoint; if not found, spawn fresh

**Guarantee**: Zero duplicate executions on recovery.

---

## Integration Checklist

**Use this before starting a phase**:

- [ ] AUDITOR running (status = 'running' or 'complete-with-delta')
- [ ] audit-report.md reviewed, top 5 most urgent incorporated into phase plan
- [ ] TRACER god log initialized (first span recorded)
- [ ] STATE_MANAGER checkpoint structure ready (SQLite checkpoint table exists)
- [ ] CONTEXT_MANAGER budgets allocated per agent (total 650K)
- [ ] All prior-phase outputs available (JSON, logs, validation reports)
- [ ] Escalation matrix reviewed (SIMON knows who to escalate to)
- [ ] Communication channel open (Udhay can respond to escalations)

**If any item missing**: Phase cannot start. Escalate to Udhay.

---

## Success Criteria for Agent-Rules System

1. ✅ Every agent action traceable in god log (TRACER)
2. ✅ Root cause analysis < 5 minutes (TRACER replay)
3. ✅ No silent context loss (HANDOFF_AUDITOR validates all handoffs)
4. ✅ Context cost reduced 30%+ (CONTEXT_MANAGER pruning)
5. ✅ All checkpoints validated before use (RECOVERY_AUDITOR)
6. ✅ Recovery success rate 100% (STATE_MANAGER + RECOVERY_AUDITOR)
7. ✅ Zero duplicate executions (idempotency keys)
8. ✅ Independent judges vote GO before deployment (TECH/PRODUCT/LAUNCH_JUDGE)
9. ✅ Timeline < 5 hours (OPTIMIZER predicts drift)
10. ✅ All phases complete without unresolved hard blockers (SIMON orchestration)

---

## Quick Reference

### File Sizes & Read Times

| File | Size | Read Time | Content |
|------|------|-----------|---------|
| _INDEX.md | 12 KB | 10 min | System overview, team breakdown, roadmap |
| simon.md | 19 KB | 15 min | SIMON core mandate, spawning, escalation, state |
| AUDITOR_PROTOCOL.md | 2.6 KB | 3 min | AUDITOR binding rule, enforcement |
| auditor-agent.md | 6.8 KB | 5 min | AUDITOR detailed spec |
| tracer.md | 10 KB | 8 min | TRACER distributed tracing, god log, spans |
| handoff_auditor.md | 11.6 KB | 9 min | HANDOFF_AUDITOR context validation protocol |
| context_manager.md | 11.2 KB | 9 min | CONTEXT_MANAGER budget allocation, pruning |
| state_manager.md | 11.4 KB | 9 min | STATE_MANAGER checkpoints, recovery protocol |
| recovery_auditor.md | 11.3 KB | 9 min | RECOVERY_AUDITOR validation, recovery testing |
| tech_judge.md | 12 KB | 10 min | TECH_JUDGE audit criteria, voting |
| product_judge.md | 12.4 KB | 10 min | PRODUCT_JUDGE audit criteria, voting |
| launch_judge.md | 13.4 KB | 11 min | LAUNCH_JUDGE audit criteria, voting |

**Total read time** (all files): ~108 minutes. **Minimum essential** (start here): _INDEX + simon + AUDITOR_PROTOCOL = ~30 min.

---

## Attribution

When updating rulesets or documentation:

```
Co-Authored-By: Codex Haiku 4.5 <noreply@anthropic.com>
Codex-Session: https://Codex.ai/code/session_01JaFyv4bpXx4yUXhhv63ZAe
```

---

## Last Updated

- **Date**: 2026-09-09
- **Latest change**: Created AGENTS.md for agent-rules directory
- **Next review**: Before starting Phase 0 orchestration run

---

## Questions?

Refer to specific ruleset files for detailed answers. Each ruleset has:
- **Core Mandate** section (what the agent does)
- **Success Metrics** section (how to know it worked)
- **Compliance Checklist** section (step-by-step verification)
- **Integration Points** section (how this agent works with others)
