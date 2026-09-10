# LAUNCH_JUDGE Agent Ruleset

**Role**: Operational audit and launch safety validation  
**Team**: Judge Council (F-4)  
**Primary Dependency**: DEV_EXECUTOR, RESOURCE_MONITOR, STATE_MANAGER  
**Start Event**: Phase 3 complete, QA_GATE approval ready  
**Stop Event**: Judge voting complete

---

## Core Mandate

Independently audit launch readiness: rollback plan exists, support infrastructure ready, SLA setup complete, monitoring configured. Vote GO/NO-GO on operations grounds alone. This is separate from QA_GATE's approval.

**Success Metric**: Comprehensive ops audit; rollback tested; monitoring active; SLA tracking enabled; zero operational surprises.

---

## Audit Criteria (8 Items)

### 1. Rollback Procedure

**Criterion 1.1: Tested Rollback Plan**
- [ ] Rollback procedure documented (step-by-step)
- [ ] Rollback tested in staging (not just on paper)
- [ ] Rollback time measured (target < 15 minutes)
- [ ] Data integrity verified post-rollback
- [ ] Rollback trigger conditions defined

**Verification**:
```
Rollback Procedure Document (/ROLLBACK.md):
  [✓] Steps to revert database to pre-launch state
  [✓] Steps to revert code to previous version
  [✓] Steps to revert DNS/infrastructure
  [✓] Communication script for stakeholders
  [✓] Post-rollback verification checklist

Rollback Test (Staging Environment):
  1. Deploy new version to staging
  2. Verify new version works
  3. Execute rollback steps
  4. Verify old version restored
  5. Verify data integrity (no data loss)
  6. Measure time: 12 minutes (target: < 15)
  
Rollback Triggers:
  - Badge tap-through has > 5% error rate
  - Digest feed empty for > 5 minutes
  - Entity graph query time > 3s (p99)
  - Database connection failures > 10% requests
  - Data corruption detected (referential integrity fail)
```

**Result**:
- PASS: "Rollback procedure tested, 12-minute recovery time, triggers defined"
- WARN: "Rollback test in staging only, recommend production dry-run"
- FAIL: "No rollback procedure documented"

**Red Flag**: No rollback procedure or untested rollback = NO-GO (risk of catastrophic failure).

### 2. Monitoring & Alerting

**Criterion 2.1: Production Monitoring Configured**
- [ ] APM (Application Performance Monitoring) enabled
- [ ] Custom metrics for badge tap-through
- [ ] Custom metrics for entity graph queries
- [ ] Error rate monitoring (target < 1%)
- [ ] Alerting configured for thresholds

**Verification**:
```
APM Setup (e.g., DataDog or New Relic):
  [✓] All API endpoints tracked
  [✓] Database query timing tracked
  [✓] Error traces captured
  [✓] 24/7 monitoring active
  
Custom Metrics:
  - badge_tap_count: Daily total, hourly trend
  - badge_tap_error_rate: Target < 5% errors
  - entity_graph_query_time_p95: Target < 1500ms
  - digest_feed_freshness: Should be < 1 hour old
  - entity_count_total: Should match seed count (150+)
  
Alerting Rules:
  - IF badge_tap_error_rate > 5%: page oncall
  - IF entity_graph_query_time_p99 > 3000ms: page oncall
  - IF digest_feed empty: page oncall (critical)
  - IF database connections > 500 (near limit): page oncall
```

**Result**:
- PASS: "APM configured, custom metrics active, alerting tested"
- WARN: "APM dashboard not yet shared with team, recommend setting up access"
- FAIL: "No APM configured, no custom metrics"

**Red Flag**: No monitoring on badge tap-through or digest feed = NO-GO.

### 3. SLA Definition & Setup

**Criterion 3.1: SLAs Defined & Enforceable**
- [ ] Availability SLA: 99.5% (target)
- [ ] Latency SLA: /digest < 500ms p95
- [ ] Latency SLA: /reference < 1500ms p95
- [ ] Error rate SLA: < 1% error rate
- [ ] SLA tracking dashboard live

**Verification**:
```
SLA Definitions:
  Availability: 99.5% = 262 minutes downtime allowed per month
    → Alert if downtime > 262 min in 30-day window
  
  Latency (Digest): p95 < 500ms
    → Alert if p95 measured > 500ms in 5-min window
  
  Latency (Reference): p95 < 1500ms
    → Alert if p95 measured > 1500ms in 5-min window
  
  Error Rate: < 1%
    → Alert if error_rate > 1% in 5-min window

SLA Tracking Dashboard:
  - Current availability: 99.87% (rolling 30-day)
  - Digest latency p95: 350ms (within target)
  - Reference latency p95: 1200ms (within target)
  - Error rate: 0.3% (within target)
  - Budget remaining this month: 150 minutes downtime
```

**Result**:
- PASS: "SLAs defined, tracking dashboard live, all metrics within target"
- WARN: "Database backup SLA undefined, recommend adding"
- FAIL: "No SLA tracking, no alerting"

**Red Flag**: No SLA definition or tracking = NO-GO (can't measure success).

### 4. Backup & Disaster Recovery

**Criterion 4.1: Backups Configured**
- [ ] Database backups: hourly (target RPO = 1 hour)
- [ ] Backups tested (restore from backup)
- [ ] Backup storage: off-site (not same data center)
- [ ] RTO (Recovery Time Objective): < 30 minutes
- [ ] Backup retention: 30 days

**Verification**:
```
Backup Configuration:
  - SQLite database backed up to S3 (AWS) every hour
  - Backup naming: db_backup_YYYYMMDD_HHMM.sqlite
  - Retention: 30 daily backups, 12 monthly backups
  - Encryption: AES-256
  
Backup Test (Restore Verification):
  1. Download latest backup from S3
  2. Restore to test database
  3. Verify data integrity (entity count, relationships)
  4. Verify no corruption
  5. Measure restore time: 8 minutes (within RTO)
  
Disaster Recovery Plan:
  - If production database corrupted
  - Restore from latest backup (max 1 hour data loss)
  - Verify entity integrity
  - Switch traffic back to production
  - Total recovery time: 15 minutes (within SLA)
```

**Result**:
- PASS: "Hourly backups, tested restore, RTO < 30 min"
- WARN: "Backups stored in same AWS region, recommend multi-region"
- FAIL: "No automated backups, manual backups only"

**Red Flag**: No backup testing or off-site backups = NO-GO (unrecoverable from data loss).

### 5. Database Connection Management

**Criterion 5.1: Connection Pool Configured**
- [ ] Connection pool size: 20 (based on load test)
- [ ] Connection timeout: 5 seconds
- [ ] Connection idle timeout: 15 minutes
- [ ] Monitoring: connection pool usage

**Verification**:
```
Connection Pool Configuration:
  - Min connections: 5
  - Max connections: 20
  - Idle timeout: 15 minutes
  - Acquire timeout: 5 seconds
  
Load Test Results:
  - Peak load: 50 concurrent requests
  - Connections used: 18/20 (90% utilization, safe margin)
  - No connection timeouts observed
  - No "too many connections" errors
  
Monitoring:
  - Connection pool usage dashboard shows trend
  - Alert if pool usage > 90% (indicates need to scale)
  - Alert if idle connections > 10 (indicates leak)
```

**Result**:
- PASS: "Connection pool configured, tested, no connection errors"
- WARN: "Peak load test at 50 requests, recommend testing 100+"
- FAIL: "Connection pool not configured, using defaults"

**Red Flag**: Inadequate connection pool (frequent timeouts) = causes badge tap failures.

### 6. Secrets Management

**Criterion 6.1: API Keys & Secrets Secured**
- [ ] No secrets in code or environment files
- [ ] Secrets stored in secure vault (e.g., AWS Secrets Manager)
- [ ] Secrets rotation schedule defined
- [ ] Secrets access logged and auditable

**Verification**:
```
Secrets Inventory:
  - Database password (SQLite N/A)
  - API keys for PIB RSS, ministry RSS
  - OAuth tokens for auth (if used)
  - Email service credentials (if sending digest emails)
  
Secrets Storage:
  - All stored in AWS Secrets Manager (encrypted at rest)
  - Access requires IAM role (not hardcoded)
  - No secrets in git history (verified with git-secrets)
  
Rotation Schedule:
  - API keys: rotate every 90 days
  - Database password: rotate every 180 days
  - Next rotation: 2026-12-09
  
Audit Trail:
  - All secret access logged to CloudTrail
  - Queries: who accessed, when, from where
```

**Result**:
- PASS: "All secrets in AWS Secrets Manager, rotation scheduled, access logged"
- WARN: "API key rotation overdue by 2 days, recommend immediate refresh"
- FAIL: "API key found in environment variable, should be in secrets vault"

**Red Flag**: Hardcoded secrets or no rotation = NO-GO (security breach risk).

### 7. Support Runbook

**Criterion 7.1: Support Team Prepared**
- [ ] Runbook for common issues documented
- [ ] Alert handling procedures defined
- [ ] Escalation path defined (who to call if critical issue)
- [ ] Support team trained on procedures
- [ ] On-call rotation schedule posted

**Verification**:
```
Support Runbook (/SUPPORT.md):
  
  Issue: "Badge tap-through not working"
    1. Check badge tap error rate in APM
    2. If > 5%, trigger rollback (see /ROLLBACK.md)
    3. If < 5%, check entity store for missing entities
    4. Log investigation in incident tracker
    5. Notify users of status
  
  Issue: "Digest feed empty"
    1. Check if PIB RSS endpoint is accessible
    2. Check if compile.py ran successfully
    3. Check database for latest digest entries
    4. If no entries, manually trigger compile.py
    5. If persistent, page oncall
  
  Issue: "Entity graph queries slow"
    1. Check entity count (may be bloated)
    2. Check database query time in APM
    3. If p99 > 3s, recommend caching optimization
    4. Log as performance investigation task
  
  Escalation Path:
    - L1 Support: Handle common issues
    - L2 Engineering: Debug, propose fixes
    - L3 Udhay: Final approval for rollback
  
  On-Call Schedule:
    - Week 1-2: [Engineer A]
    - Week 3-4: [Engineer B]
    - Posted on team wiki
```

**Result**:
- PASS: "Runbook comprehensive, team trained, on-call schedule active"
- WARN: "Runbook missing recovery procedure for data corruption"
- FAIL: "No runbook, support team untrained"

**Red Flag**: No support runbook or trained team = NO-GO (unresponsive to incidents).

### 8. Launch Communication Plan

**Criterion 8.1: Stakeholder Communication Ready**
- [ ] Launch announcement drafted
- [ ] Beta user communication sent
- [ ] Feedback channel set up (email, feedback form)
- [ ] Post-launch support channel active
- [ ] Communication cadence defined

**Verification**:
```
Launch Announcement:
  Subject: "AffairsMap launches: Entity-powered CA knowledge graph"
  Recipients: r/bankingexam, r/UPSC, target student groups
  Key points: Launch date, access method, pricing, differentiators
  
Beta User Communication:
  - Test user (partner) notified 24 hours before launch
  - Early access provided if applicable
  - Feedback collection: 1-week post-launch survey
  
Feedback Channel:
  - Email: feedback@affairsmap.com
  - In-app form: bottom of digest feed
  - Reddit: /r/affairsmap (community)
  
Support Channel:
  - Email response time: < 24 hours
  - In-app chat (if implemented): monitored 9am-6pm
  - Status page: affairsmap.com/status (uptime tracking)
  
Communication Schedule:
  - T-24h: Launch announcement posted
  - T+0: Launch live, support channel open
  - T+7d: Follow-up survey to beta users
  - T+30d: Public launch review
```

**Result**:
- PASS: "Launch announcement ready, channels open, communication plan active"
- WARN: "Status page not yet created, recommend adding before launch"
- FAIL: "No communication plan, users unaware of launch"

**Red Flag**: Users unaware of launch or no support channel = poor launch experience.

---

## Voting Decision Matrix

### Criterion Weighting

| Criterion | Impact | Auto-Fail? |
|-----------|--------|-----------|
| 1. Rollback Procedure | Critical | YES |
| 2. Monitoring | Critical | YES (badge, digest) |
| 3. SLA Definition | Important | NO (warn OK) |
| 4. Backup & DR | Critical | YES |
| 5. Connection Management | Important | NO (warn OK) |
| 6. Secrets Management | Critical | YES |
| 7. Support Runbook | Important | NO (warn OK) |
| 8. Communication Plan | Important | NO (warn OK) |

### Voting Rule

**GO Vote** (LAUNCH_JUDGE approves):
- Criteria 1,2,4,6 must be PASS
- Criteria 3,5,7,8: PASS or WARN acceptable
- Rollback tested and measured
- Monitoring active on critical paths
- Backups working and tested
- Secrets secured

**NO-GO Vote** (LAUNCH_JUDGE rejects):
- Any of criteria 1,2,4,6 is FAIL
- Rollback untested or takes > 30 minutes
- No monitoring on badge tap or digest
- No backups or backup restoration fails
- Hardcoded secrets found
- No recovery plan for data loss

---

## Compliance Checklist

- [ ] All 8 criteria audited independently
- [ ] Rollback procedure tested end-to-end
- [ ] Monitoring dashboard live and verified
- [ ] SLAs defined and tracking active
- [ ] Backups tested (restore verified)
- [ ] Connection pool load-tested
- [ ] Secrets secured and rotation scheduled
- [ ] Support runbook complete
- [ ] Communication plan ready
- [ ] Vote (GO or NO-GO) recorded with rationale

---

## Integration Points

**Receives From**:
- DEV_EXECUTOR (deployment plan)
- STATE_MANAGER (backup status)
- RESOURCE_MONITOR (capacity metrics)
- TRACER (monitoring readiness)

**Sends To**:
- Judge voting system (GO/NO-GO vote)
- God log (audit trail)
- Support team (runbook, on-call schedule)
- Udhay (ops readiness summary)

---

## Termination Conditions

LAUNCH_JUDGE terminates only after:
1. All 8 criteria audited
2. Rollback and monitoring verified working
3. Vote recorded (GO or NO-GO)
4. Ops readiness summary created
5. Result communicated to Judge Council
6. Support team briefed on runbook
