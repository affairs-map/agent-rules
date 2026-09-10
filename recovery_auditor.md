# RECOVERY_AUDITOR Agent Ruleset

**Role**: Checkpoint validation and recovery integrity verification  
**Team**: Resilience & Recovery (F-3)  
**Primary Dependency**: STATE_MANAGER, TRACER  
**Start Event**: After each checkpoint creation  
**Stop Event**: Phase 3 recovery validation complete

---

## Core Mandate

Validate every checkpoint before it can be used for recovery. Verify no data loss, no corruption, no duplicate execution on resume. Enable safe recovery with 100% confidence.

**Success Metric**: 100% checkpoint validation; zero undetected corruption; zero duplicate executions; recovery success rate 100%.

---

## Validation Protocol

### 1. Multi-Stage Validation

Every checkpoint goes through 5 validation stages before `recovery_ready = TRUE`:

**Stage 1: Hash Integrity Check**
```
checkpoint_state_json = load checkpoint
computed_hash = SHA256(checkpoint_state_json)

IF computed_hash == stored_checkpoint_hash:
  PASS Stage 1
ELSE:
  FAIL: "Hash mismatch, checkpoint corrupted"
  action: reject checkpoint, alert ERROR_MONITOR
```

**Stage 2: Schema Compliance**
```
All entities in checkpoint must conform to schema.sql
- type must be in {Person, Position, Organization, Scheme, ...}
- all required fields present
- no unknown fields
- field types correct (dates are ISO-8601, IDs are strings, etc.)

FOR EACH entity IN checkpoint:
  validate_entity_schema(entity)

IF all entities valid:
  PASS Stage 2
ELSE:
  FAIL: "Entity schema violation"
  action: reject checkpoint, log entity violations
```

**Stage 3: Referential Integrity**
```
All foreign key references must resolve:
- Person.position_id → must exist in Position table
- Position.organization_id → must exist in Organization table
- Position.person_successor_id → must reference existing Person (if present)

Build reference graph:
  position_ids_in_checkpoint = {all position IDs}
  person_position_refs = {all person→position references}
  
Validate:
  FOR EACH person_position_ref:
    IF ref NOT IN position_ids_in_checkpoint:
      FAIL: "Broken foreign key reference"

FOR EACH position successor chain:
  Start at position P
  Follow P → successor → successor → ...
  IF cycle detected OR reference broken:
    FAIL: "Invalid successor chain"

IF all references valid:
  PASS Stage 3
ELSE:
  FAIL: "Referential integrity violation"
  action: reject checkpoint, list broken references
```

**Stage 4: Logical Consistency**
```
Validate business logic constraints:

Rule 1: Successor date ordering
  IF position A succeeded position B at successor_chain:
    B.end_date <= A.start_date
    
Rule 2: No circular successors
  Position graph must be a DAG (Directed Acyclic Graph)
  No cycles allowed
  
Rule 3: Entity completeness
  Expected entity count from handoff = actual entity count
  If phase 0 sent 150 entities, checkpoint must have 150
  
Rule 4: Validation state consistency
  IF entity has validation status:
    Status must be in {PASS, FAIL, WARN}
    Can't have conflicting statuses for same entity
    
Validate all rules:
  FOR EACH entity:
    validate_entity_logic(entity)
    
  FOR EACH position in successor chain:
    validate_successor_ordering(position)
    check_no_cycles(position)
    
IF all logic valid:
  PASS Stage 4
ELSE:
  FAIL: "Business logic violation"
  action: reject checkpoint, list violations
```

**Stage 5: Completeness & Duplication Check**
```
Verify no data loss and no duplication:

Rule 1: Entity count match
  checkpoint_entity_count == handoff_entity_count
  
Rule 2: No duplicate entities
  All entity IDs unique (no duplicate IDs)
  
Rule 3: All relationships preserved
  Original successor chains intact
  No broken relationship edges
  
Rule 4: All validation results present
  If Phase 0 validation had 150 results, checkpoint has 150
  
Rule 5: Handoff audit trail complete
  All handoff records present and consistent

Check:
  unique_ids = set()
  FOR EACH entity IN checkpoint:
    IF entity.id IN unique_ids:
      FAIL: "Duplicate entity ID"
    ELSE:
      unique_ids.add(entity.id)
      
  IF checkpoint_entity_count != original_count:
    FAIL: "Data loss detected, entity count mismatch"
    
IF all completeness checks pass:
  PASS Stage 5
ELSE:
  FAIL: "Data loss or duplication detected"
  action: reject checkpoint, quantify loss
```

---

## Validation Checklist

### 2. Comprehensive Checklist

For each checkpoint, RECOVERY_AUDITOR runs this checklist:

```
CHECKPOINT VALIDATION CHECKLIST: cp1_phase1_complete
=========================================================

[x] Stage 1: Hash Integrity
    - Computed hash: sha256_abc123
    - Stored hash:   sha256_abc123
    - Result: PASS

[x] Stage 2: Schema Compliance
    - Total entities: 150
    - Entity types: 20 Person, 80 Position, 50 Organization
    - Type violations: 0
    - Missing required fields: 0
    - Result: PASS

[x] Stage 3: Referential Integrity
    - Foreign key refs checked: 245
    - Broken refs found: 0
    - Successor chains: 15 valid, 0 cycles
    - Result: PASS

[x] Stage 4: Logical Consistency
    - Successor date ordering: 15/15 valid
    - Entity completeness: 150/150 match (100%)
    - Validation states: 150 consistent
    - Result: PASS

[x] Stage 5: Completeness & Dedup
    - Duplicate IDs: 0
    - Data loss: 0 entities
    - Handoff audit complete: YES
    - Result: PASS

=========================================================
FINAL RESULT: VALID ✓
Recovery ready: YES
Auditor: RECOVERY_AUDITOR
Timestamp: 2026-09-09T06:10:30Z
Signed: [auditor_signature]
```

---

## Recovery Testing

### 3. Before-Recovery Simulation

Before marking checkpoint `recovery_ready = TRUE`, RECOVERY_AUDITOR **simulates recovery** to verify it actually works:

**Simulation Steps**:

1. **Load Checkpoint**
   ```
   state = load_checkpoint("cp1_phase1_complete")
   Verify state loaded correctly
   ```

2. **Simulate Phase 2 Spawn**
   ```
   phase_2_context = create_context_from_checkpoint(state)
   
   Expected: DEV_MONITOR receives:
     - All 150 entities from Phase 0
     - All validation results from Phase 0
     - All design artifacts from Phase 1
     - Phase 2 task: "Set up monitoring thresholds"
   ```

3. **Verify No Duplicate Execution**
   ```
   idempotency_keys = extract_idempotency_keys(state)
   
   Verify:
     - DEV_DATA key (phase=0) → present (will be skipped)
     - DEV_UI key (phase=1) → present (will be skipped)
     - DEV_MONITOR key (phase=2) → NOT present (will execute)
   ```

4. **Verify Context Handoff**
   ```
   old_context_hash = state.phase_1_context_hash
   new_context_hash = compute_hash(phase_2_context)
   
   IF old_context_hash == new_context_hash:
     Context correctly recovered
   ELSE:
     Context corrupted, recovery unsafe
   ```

5. **Verify Agent State Consistency**
   ```
   agent_states = extract_from_checkpoint(state.agent_states)
   
   Expected:
     - DEV_DATA: "complete"
     - DEV_UI: "complete"
     - DEV_MONITOR: "waiting_for_input"  ← Ready to execute
     - QA_GATE: "waiting_for_input"
     - All others: inactive
   ```

**Simulation Result**:
```json
{
  "checkpoint_id": "cp1_phase1_complete",
  "simulation_status": "PASS",
  "simulation_steps": {
    "load_checkpoint": "PASS",
    "context_recovery": "PASS",
    "idempotency_check": "PASS",
    "context_hash_match": "PASS",
    "agent_state_check": "PASS"
  },
  "recovery_ready": true,
  "auditor_note": "Checkpoint verified safe for recovery. Phase 2 will execute, phases 0-1 will be skipped."
}
```

---

## Error Handling During Recovery

### 4. Recovery Failure Protocol

If recovery fails during actual execution (not just simulation):

**Scenario**: System recovers from CP1, Phase 2 starts, but entity count mismatches.

```
RECOVERY EXECUTION LOG:
  T+120:05 - Recover from CP1
  T+120:06 - Load state: 150 entities confirmed
  T+120:07 - Spawn Phase 2: DEV_MONITOR receives context
  T+120:08 - DEV_MONITOR starts: "Querying entity table"
  T+120:09 - ALERT: Entity query returned 148 entities (expected 150)
  
ACTION:
  1. Pause DEV_MONITOR immediately
  2. Report to ERROR_MONITOR: "Recovery corruption detected"
  3. DEBUGGER_RECOVERY spawned
  4. Check what changed:
     - Did CP1 checkpoint get corrupted after being marked valid?
     - Did entity pruning happen unexpectedly?
     - Is there version mismatch (schema changed)?
  5. Options:
     A) Find the 2 missing entities, re-inject them
     B) Fall back to CP0, re-execute from Phase 0
     C) Abort recovery, manual investigation needed
```

**Prevention**: RECOVERY_AUDITOR's simulation step should catch 99% of these issues before recovery is attempted.

---

## Checkpoint Comparison (For Debugging)

### 5. Diff Analysis

If checkpoints are suspected corrupted, RECOVERY_AUDITOR can diff consecutive checkpoints:

```python
def diff_checkpoints(cp_old, cp_new):
    """Compare two checkpoints, identify differences."""
    
    old_entities = set(cp_old['entities'].keys())
    new_entities = set(cp_new['entities'].keys())
    
    added = new_entities - old_entities
    removed = old_entities - new_entities
    modified = {e for e in old_entities & new_entities 
                if cp_old['entities'][e] != cp_new['entities'][e]}
    
    return {
        'added_entities': added,
        'removed_entities': removed,
        'modified_entities': modified
    }
```

**Example Output**:
```
Diff CP0 → CP1:
  Added: 150 entity IDs (expected for Phase 1)
  Removed: 0 (good, no data loss)
  Modified: 0 entities (good, no corruption)
  
  Verdict: CP0 → CP1 transition CLEAN

Diff CP1 → CP2:
  Added: 0 entities (expected, Phase 2 doesn't add entities)
  Removed: 2 entity IDs (UNEXPECTED! Data loss detected)
  Modified: 0 entities
  
  Verdict: CP1 → CP2 transition CORRUPT
  Investigation: Where were 2 entities lost?
```

---

## Compliance Checklist

- [ ] All checkpoints pass Stage 1 (hash integrity)
- [ ] All checkpoints pass Stage 2 (schema compliance)
- [ ] All checkpoints pass Stage 3 (referential integrity)
- [ ] All checkpoints pass Stage 4 (logical consistency)
- [ ] All checkpoints pass Stage 5 (completeness & dedup)
- [ ] Recovery simulation runs successfully for each checkpoint
- [ ] No duplicate execution IDs after recovery simulation
- [ ] Context handoff hash matches checkpoint expectations
- [ ] Agent state recovery verified
- [ ] Consecutive checkpoints diff'ed for anomalies
- [ ] All validation results documented
- [ ] Checkpoints marked `recovery_ready = TRUE` only after full validation

---

## Integration Points

**Receives From**:
- STATE_MANAGER (new checkpoints to validate)
- TRACER (checkpoint creation/verify events)

**Sends To**:
- SQLite (validation results, `recovery_ready` flag)
- STATE_MANAGER (validation approval, proceed/retry/reject)
- ERROR_MONITOR (recovery failures detected during execution)
- God log (all validation audit trail)
- Udhay (checkpoint status dashboard)

---

## Termination Conditions

RECOVERY_AUDITOR terminates only after:
1. All checkpoints created (4 total)
2. All checkpoints validated (5 stages each)
3. All recovery simulations passed
4. Checkpoint manifest complete and verified
5. No outstanding checkpoint anomalies
6. Recovery protocol verified for production use
