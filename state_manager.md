# STATE_MANAGER Agent Ruleset

**Role**: System state persistence and crash recovery  
**Team**: Resilience & Recovery (F-3)  
**Primary Dependency**: TRACER, HANDOFF_AUDITOR  
**Start Event**: Phase 0 T+0:00  
**Stop Event**: Phase 3 complete + recovery validation

---

## Core Mandate

Checkpoint system state at phase boundaries. Enable resume from last checkpoint if SIMON or any critical agent restarts. Eliminate data loss and 90% task failure risk from unplanned interruptions.

**Success Metric**: 100% checkpoint creation; zero data loss on recovery; 0% duplicate execution; recovery time < 5 minutes.

---

## Checkpoint Strategy

### 1. Checkpoint Lifecycle

**Checkpoint Points** (4 total per run):

| Checkpoint | Timing | State Captured | Storage |
|------------|--------|----------------|---------|
| CP0 | T+0 (Phase 0 start) | Initial SIMON state, spawn config | `/checkpoints/cp0_phase0_start.json` |
| CP1 | T+90 (Phase 1 complete) | All Phase 0 outputs + validation results | `/checkpoints/cp1_phase1_complete.json` |
| CP2 | T+150 (Phase 2 complete) | All Phase 1 outputs + UI artifacts | `/checkpoints/cp2_phase2_complete.json` |
| CP3 | T+210 (Phase 3 complete) | All Phase 2 outputs + deployment ready state | `/checkpoints/cp3_phase3_complete.json` |

### 2. What Gets Checkpointed

Each checkpoint is a **complete state snapshot**:

```json
{
  "checkpoint_id": "cp1_phase1_complete",
  "timestamp_ms": 1694265400000,
  "phase_completed": 0,
  "state_snapshot": {
    "entities": {...all 150 entities from Phase 0...},
    "entity_relationships": {...successor chains, position history...},
    "validation_results": {
      "schema_validation": "PASS",
      "entity_count": 150,
      "failures": []
    },
    "phase_0_summary": "Phase 0 complete: 150 entities ingested, all validated",
    "agent_states": {
      "DEV_DATA": "complete",
      "DEV_UI": "waiting_for_input",
      "DEV_MONITOR": "waiting_for_input"
    }
  },
  "handoff_status": {
    "phase_0_to_1_handoff": {
      "status": "complete",
      "entities_handed_off": 150,
      "validation_passed": true,
      "timestamp": 1694265400000
    }
  },
  "checkpoint_hash": "sha256_abc123",
  "checkpoint_size_bytes": 5242880,
  "verified_by": "RECOVERY_AUDITOR",
  "recovery_ready": true
}
```

**Size Management**:
- CP0: ~1 MB (startup config only)
- CP1: ~5 MB (Phase 0 entity data + validation)
- CP2: ~8 MB (Phase 1 mockups + design tokens)
- CP3: ~6 MB (Phase 2 deployment scripts + rollback plan)
- **Total**: ~20 MB for 4 checkpoints (reasonable for SQLite backup)

### 3. Storage Location

All checkpoints stored in SQLite checkpoint table:

```sql
CREATE TABLE checkpoints (
  checkpoint_id TEXT PRIMARY KEY,
  phase_number INT,
  timestamp_ms BIGINT,
  state_json TEXT,               -- Full state as JSON
  checkpoint_hash TEXT,          -- SHA256 of state_json
  checkpoint_size_bytes INT,
  verified_by TEXT,              -- RECOVERY_AUDITOR verification
  recovery_ready BOOLEAN,
  created_at DATETIME DEFAULT NOW(),
  
  UNIQUE(phase_number, timestamp_ms)
);
```

**Why SQLite**: Atomic writes, built-in ACID compliance, portable, queryable.

---

## Recovery Protocol

### 4. Detection & Activation

**Recovery Trigger** (automatically detected):
```
IF SIMON restarts OR any critical agent crashes:
  DETECTION: Recovery system active
  ACTION: Load last checkpoint
  VALIDATION: Verify checkpoint integrity
  RESUME: Continue from checkpoint point
```

**Example Scenario**:
- T+120 min: Phase 2 executing (DEV_MONITOR processing SLA metrics)
- INTERRUPT: Infrastructure restart (routine container recycle)
- T+120:05 min: System reboots, SIMON cold starts
- DETECTION: "No active execution state found. Check for recovery checkpoints."
- RECOVERY: Load CP1 (Phase 1 complete checkpoint from T+90)
- RESUME: Start Phase 2 execution from T+90 checkpoint (not from T+0)
- **Time Saved**: 90 minutes re-execution eliminated

### 5. Checkpoint Loading

When system restarts:

```python
def recover_from_checkpoint():
    # Find latest successful checkpoint
    latest_cp = db.query("""
        SELECT * FROM checkpoints 
        WHERE recovery_ready = TRUE 
        ORDER BY timestamp_ms DESC LIMIT 1
    """)
    
    if not latest_cp:
        raise RecoveryError("No valid checkpoint found")
    
    # Verify integrity before loading
    stored_hash = latest_cp['checkpoint_hash']
    computed_hash = SHA256(latest_cp['state_json'])
    
    if stored_hash != computed_hash:
        raise IntegrityError("Checkpoint corrupted, recovery impossible")
    
    # Load state into memory
    global_state = json.loads(latest_cp['state_json'])
    
    # Resume execution from checkpoint phase + 1
    phase_to_resume = latest_cp['phase_number'] + 1
    
    return phase_to_resume, global_state
```

**Safety**: Integrity check (hash comparison) ensures no corrupted data loaded.

---

## No Duplicate Execution

### 6. Idempotency Guarantee

Critical: **Do not re-execute completed work.**

When resuming from CP1 (Phase 1 complete):

```
Phase 0: [SKIP - already completed, in checkpoint]
Phase 1: [SKIP - already completed, in checkpoint]
Phase 2: [EXECUTE - resume from beginning]
Phase 3: [EXECUTE - when ready]
```

**Idempotency Key** per agent:

```
DEV_DATA idempotency_key = hash(phase=0, timestamp=T+0)
  If this key exists in checkpoint, DEV_DATA SKIPS.
  
DEV_MONITOR idempotency_key = hash(phase=2, timestamp=T+150)
  If this key NOT in checkpoint, DEV_MONITOR executes.
```

Database schema:

```sql
CREATE TABLE idempotency_keys (
  idempotency_key TEXT PRIMARY KEY,
  agent_id TEXT,
  phase_number INT,
  execution_timestamp_ms BIGINT,
  status TEXT,  -- 'completed', 'failed', 'skipped_on_recovery'
  created_at DATETIME DEFAULT NOW()
);
```

When DEV_DATA starts:
```
IF db.exists(idempotency_key(phase=0)) AND recovery_mode:
  SKIP execution, log "Recovered from checkpoint"
ELSE:
  EXECUTE normally
```

---

## Checkpoint Verification

### 7. RECOVERY_AUDITOR Validation

Before marking checkpoint as `recovery_ready = TRUE`, RECOVERY_AUDITOR validates:

**Validation 1: Schema Integrity**
```
All entities in checkpoint must:
- Have valid type (Person, Position, Organization, etc.)
- Have all required fields per entity type
- Pass foreign key constraints
- Have consistent state (no orphaned references)
```

**Validation 2: Hash Consistency**
```
SHA256(checkpoint state) must match stored checkpoint_hash
Any mismatch → reject checkpoint, mark recovery_unsafe
```

**Validation 3: Logical Consistency**
```
If entity A "succeeded" entity B at position P:
- B's end_date <= A's start_date
- P must exist in entity_store
- No circular successions

If these fail → reject checkpoint
```

**Validation 4: Completeness**
```
All entities from Phase 0 must be present
All validation results must be present
All handoff confirmations must be present

If missing any → reject checkpoint, mark incomplete
```

**Validation Result**:
```json
{
  "checkpoint_id": "cp1_phase1_complete",
  "validation_status": "PASS",
  "checks_passed": 4,
  "checks_failed": 0,
  "issues": [],
  "recovery_ready": true,
  "auditor_timestamp": 1694265420000
}
```

If any check fails: `recovery_ready = FALSE`, do not load this checkpoint.

---

## Partial Recovery (If Latest Checkpoint Bad)

### 8. Fallback Strategy

If latest checkpoint fails validation:

```
CP3 fails validation → try CP2
CP2 fails validation → try CP1
CP1 fails validation → try CP0
CP0 fails validation → abort, full re-execution required
```

**Example**: At T+180, system detects CP2 is corrupted.
- Action: Use CP1 (last good checkpoint at T+90)
- Cost: Re-execute Phase 2 only (~60 min)
- Benefit: Saved 90 min from Phase 0+1 re-execution

---

## Checkpoint Manifest

### 9. Recovery Audit Trail

Maintain manifest file tracking all checkpoints:

```json
{
  "manifest_version": "1.0",
  "session_id": "session_20260909_affairsmap",
  "checkpoints": [
    {
      "checkpoint_id": "cp0_phase0_start",
      "phase": 0,
      "status": "valid",
      "created_at": "2026-09-09T05:40:00Z",
      "verified_at": "2026-09-09T05:40:30Z",
      "size_bytes": 1048576,
      "hash": "sha256_cp0"
    },
    {
      "checkpoint_id": "cp1_phase1_complete",
      "phase": 1,
      "status": "valid",
      "created_at": "2026-09-09T06:10:00Z",
      "verified_at": "2026-09-09T06:10:30Z",
      "size_bytes": 5242880,
      "hash": "sha256_cp1"
    },
    {...}
  ],
  "last_valid_checkpoint": "cp1_phase1_complete",
  "recovery_mode": false,
  "last_recovery_attempt": null
}
```

This manifest enables Udhay to:
- See which checkpoints are available
- Identify which was last valid
- Track recovery history if restarts occur

---

## Checkpoint Creation Timeline

### 10. Automation

At each checkpoint time, STATE_MANAGER automatically:

```
T+90:00 (Phase 1 complete):
  01. Collect all Phase 0/1 outputs
  02. Serialize to JSON
  03. Compute SHA256 hash
  04. Write to checkpoints table (atomic write)
  05. Call RECOVERY_AUDITOR.validate_checkpoint(cp1)
  06. Wait for RECOVERY_AUDITOR confirmation
  07. If validation passes: mark recovery_ready = TRUE
  08. If validation fails: investigate, don't mark ready
  09. Log to god log (TRACER)
  10. Report success to SIMON

T+150:00 (Phase 2 complete):
  [Repeat steps 01-10 for cp2]

T+210:00 (Phase 3 complete):
  [Repeat steps 01-10 for cp3]
```

**Time Budget**: Checkpoint creation/verification takes ~30 seconds per checkpoint. 4 checkpoints = 2 min total (acceptable overhead).

---

## Disaster Recovery (Unrecoverable Corruption)

### 11. Last Resort

If all checkpoints corrupted and recovery impossible:

```
SCENARIO: System unrecoverable at T+120
ACTION: Full restart from T+0
COST: 120 minutes re-execution (same as first run)
NOTIFICATION: ERROR_MONITOR reports to Udhay
  "All checkpoints failed validation. Recovery impossible."
  "Recommend: Full restart from Phase 0."
  "New ETA: T+2:15 (vs T+1:15 if recovery had worked)"

DECISION: Udhay decides
  Option A: Do full restart now
  Option B: Investigate corruption, fix, resume from last-best checkpoint
  Option C: Abort launch, defer to next day
```

---

## Compliance Checklist

- [ ] All 4 checkpoints created at scheduled times
- [ ] Each checkpoint < 10 MB in size
- [ ] Checkpoint hashes computed and stored
- [ ] RECOVERY_AUDITOR validates each checkpoint
- [ ] All checkpoints marked `recovery_ready = TRUE`
- [ ] Idempotency keys tracked for all agents
- [ ] Recovery protocol tested (simulated restart)
- [ ] No duplicate execution on recovery
- [ ] Manifest file accurate and complete
- [ ] Fallback to earlier checkpoint works
- [ ] Unrecoverable corruption path documented
- [ ] Udhay can query recovery status anytime

---

## Integration Points

**Receives From**:
- SIMON (phase completion events)
- All agents (state data for checkpointing)
- RECOVERY_AUDITOR (validation results)
- TRACER (checkpoint create/verify events)

**Sends To**:
- SQLite (checkpoint storage)
- RECOVERY_AUDITOR (checkpoint for validation)
- God log (checkpoint audit trail)
- Udhay (recovery status, manifest)

---

## Termination Conditions

STATE_MANAGER terminates only after:
1. Phase 3 complete
2. CP3 created and verified
3. Manifest finalized
4. All checkpoints marked recovery_ready or investigation complete
5. Recovery protocol documented for next run
