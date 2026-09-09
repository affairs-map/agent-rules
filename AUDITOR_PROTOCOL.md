# AUDITOR Protocol — Binding Enforcement Rule

**RULE: AUDITOR must spawn at T+0:00 of every session. No exceptions.**

(Full content - moved from affairsmap-pipeline/)

## Session Initialization (Mandatory Pre-Phase 0)

Every session MUST follow this sequence:

### T+0:00 — AUDITOR Spawn
SIMON verifies: Is AUDITOR running?
- If no → Spawn AUDITOR immediately
- If yes → Skip spawn (already running)

### T+5:00 — AUDITOR Reports
- audit-report.md generated
- Top 5 Most Urgent listed
- CRITICAL issues flagged

### T+6:00 — SIMON Reviews
- SIMON reads audit-report.md
- Extracts Top 5 Most Urgent
- Incorporates into Phase 0 plan

### T+8:00 — Phases Begin
- Phase 0 includes high-priority fixes
- AUDITOR continues scanning in background

---

## AUDITOR Continuous Updates (Every 30 Min)

During phases, AUDITOR:
- Scans for NEW issues (not in initial report)
- Marks fixed issues ✅ FIXED (T+[time])
- Updates audit-report.md with delta
- SIMON checks report at status points
- If critical new issue found → SIMON escalates to Udhay

---

## Enforcement Mechanism

**Storage:** automation_status table
```sql
INSERT INTO automation_status (phase, status, last_updated, notes) 
VALUES ('auditor-continuous', 'running', NOW(), 'AUDITOR spawned T+0:00');
```

**Verification:** SIMON queries before Phase 1:
```sql
SELECT status FROM automation_status WHERE phase='auditor-continuous';
-- Should return: 'running' or 'complete-with-delta'
-- If 'pending' or 'missing': ESCALATE TO UDHAY immediately
```

**Binding Rule:** If AUDITOR status is NOT 'running' when Phase 1 tries to spawn → **BLOCK Phase 1, escalate to Udhay.**

---

## Route-Health Findings → Hard QA_GATE Block

Any CRITICAL-severity route-health finding (e.g., "404 routes found: /organizations/reserve-bank-of-india returns 404") blocks QA_GATE and judge approval:

**Gate Rule:**
- QA_GATE cannot approve while audit-report.md has UNRESOLVED CRITICAL route-health findings
- Judges (TECH_JUDGE, PRODUCT_JUDGE, LAUNCH_JUDGE) must each verify: "Route-health status in audit-report: CRITICAL issues = none, or all resolved"
- If any CRITICAL route-health issue remains open → All judges vote NO-GO, regardless of other criteria

**Resolution:**
- Route-health finding is marked FIXED only when the broken route is either (a) deployed with a working implementation, or (b) confirmed dead and removed from compiled data (not silent/unhandled)

---

## What Gets Audited (Continuous Scan)

### Every 10 Min:
- Build errors (npm run build)
- TypeScript errors (tsc --noEmit)
- Missing files (referenced but not found)
- 404 routes (dead links in compiled data)

### Every 30 Min:
- New TODOs/FIXMEs in code
- Orphaned entities (broken FK refs)
- Empty data files (ambassadors.json, topics.json, etc.)
- Missing route implementations
- Component rendering failures

### Every 60 Min:
- Performance regressions (page load times)
- SEO gaps (missing schemas, metadata)
- Accessibility violations (a11y)
- Test coverage drops
- Dependency updates needed

---

## Success Criteria

AUDITOR is working if:
- ✅ audit-report.md exists + updated every 30 min
- ✅ Top 5 Most Urgent always listed first
- ✅ Fixed issues marked ✅ FIXED (T+[time])
- ✅ New issues added to report as discovered
- ✅ Critical escalations reach Udhay <5 min after discovery
- ✅ No phase starts without AUDITOR status = 'running'
