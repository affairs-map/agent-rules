# TRACER Agent Ruleset

**Role**: Distributed tracing and execution observability  
**Team**: Observability & Traceability (F-2)  
**Primary Dependency**: All agents (cross-cutting concern)  
**Start Event**: Phase 0 T+0:00  
**Stop Event**: Phase 3 complete, post-launch verification done

---

## Core Mandate

Capture every agent action, tool call, and state transition as structured spans compliant with OpenTelemetry standard. Maintain a write-once-read-many god log enabling root cause analysis, incident replay, and audit trails.

**Success Metric**: 100% of agent actions traceable via unified god log; root cause analysis time < 5 minutes for any failure event.

---

## Span Capture Protocol

### 1. Span Types

**Agent Spawn Span**
- When: SIMON spawns any agent
- Captures: Agent name, ruleset version, spawn time, input payload hash
- Parent: SIMON_ORCHESTRATION_SPAN
- Attributes: `agent_id`, `role_type`, `spawn_timestamp_ms`, `input_payload_hash`

**Tool Call Span**
- When: Agent invokes a tool (e.g., `compile.py`, `entity_store.py`, `web_search`)
- Captures: Tool name, version, parameters, call time, result status
- Parent: Agent's active span
- Attributes: `tool_name`, `tool_version`, `params_hash`, `call_duration_ms`, `result_status` (success/failure/timeout)

**State Transition Span**
- When: Agent internal state changes (e.g., "validation_pending" → "validation_pass")
- Captures: State name, previous state, transition time, reason
- Parent: Agent's active span
- Attributes: `prev_state`, `new_state`, `transition_reason`, `timestamp_ms`

**Handoff Span**
- When: One agent hands off context to another
- Captures: Sender agent, receiver agent, context packet size, content hash, handoff time
- Parent: Sender agent's span
- Attributes: `sender_id`, `receiver_id`, `packet_size_bytes`, `content_hash_sha256`, `handoff_timestamp_ms`

**Validation Span**
- When: Any QA/Judge agent validates output
- Captures: Validator name, validation type, pass/fail, criteria evaluated
- Parent: Validated agent's span
- Attributes: `validator_id`, `validation_type`, `result` (pass/fail/warn), `criteria_count`, `duration_ms`

**Error Span**
- When: Any failure or exception occurs
- Captures: Error type, message, stack trace, recovering agent (if any)
- Parent: The agent that errored
- Attributes: `error_type`, `error_message`, `stack_trace`, `recovery_attempted`, `timestamp_ms`

### 2. Span Hierarchy

All spans maintain parent-child relationships to preserve causal chains:

```
SIMON_ORCHESTRATION_SPAN (root, T+0:00 start)
├── Phase 0 Span
│   ├── DEV_DATA Agent Spawn
│   │   ├── Entity Store Write Tool Call
│   │   ├── State Transition (pending→executing)
│   │   ├── Validation Span (QA_DATA)
│   │   └── Handoff Span (to DEV_UI)
│   ├── DEV_UI Agent Spawn
│   ├── DEV_MONITOR Agent Spawn
│   └── Phase 0→1 Handoff Verification
├── Phase 1 Span
│   └── ...
└── [Continue for all phases]
```

This hierarchy allows filtering by phase, agent, tool, or error type.

---

## God Log Storage

### 1. God Log Schema

**Write-Once Storage** (SQLite append-only table):

```sql
CREATE TABLE god_log (
    span_id TEXT PRIMARY KEY,              -- UUID
    parent_span_id TEXT,                   -- NULL if root
    span_type TEXT,                        -- 'agent_spawn', 'tool_call', etc.
    timestamp_ms BIGINT,                   -- Milliseconds since epoch
    agent_id TEXT,                         -- Agent that created this span
    attributes JSON,                       -- All span attributes as JSON
    created_at DATETIME DEFAULT NOW(),     -- Immutable creation time
    FOREIGN KEY(parent_span_id) REFERENCES god_log(span_id)
);
```

**Immutability Rule**: Once a row is written, it is never updated or deleted. All corrections are new spans with a reference to the original (error correction via new entry, not overwrite).

### 2. God Log Entry Format

Every entry follows this structure:

```json
{
  "span_id": "span_abc123def456",
  "parent_span_id": "span_xyz789",
  "span_type": "tool_call",
  "timestamp_ms": 1694264400000,
  "agent_id": "DEV_DATA",
  "attributes": {
    "tool_name": "compile.py",
    "tool_version": "3.2.1",
    "params_hash": "sha256_abc123",
    "call_duration_ms": 45000,
    "result_status": "success",
    "output_size_bytes": 2048000
  }
}
```

### 3. Checkpointing

At each phase boundary (T+0, T+90, T+150, T+210), TRACER exports god_log to immutable file:

```
/logs/god_log_phase_0_complete_t0.jsonl
/logs/god_log_phase_1_complete_t90.jsonl
/logs/god_log_phase_2_complete_t150.jsonl
/logs/god_log_phase_3_complete_t210.jsonl
```

Each file is checksummed (SHA-256) and archived. No modifications after checkpoint.

---

## Distributed Tracing (OpenTelemetry Compliance)

### 1. Trace Headers

Every inter-agent message includes OpenTelemetry headers:

```
traceparent: 00-[trace_id_32hex]-[span_id_16hex]-01
tracestate: affairsmap=phase:0,agent:simon
```

Agents propagate these headers on all downstream calls, preserving the trace ID across the entire workflow.

### 2. Span Export Format

TRACER exports traces in OTLP JSON format every 5 minutes:

```json
{
  "resourceSpans": [
    {
      "resource": {
        "attributes": {
          "service.name": "affairsmap-orchestrator",
          "service.version": "1.0.0"
        }
      },
      "scopeSpans": [
        {
          "scope": {
            "name": "affairsmap.tracer"
          },
          "spans": [
            {
              "traceId": "trace_xyz",
              "spanId": "span_abc",
              "parentSpanId": "span_parent",
              "name": "DEV_DATA.compile",
              "kind": 1,
              "startTimeUnixNano": 1694264400000000000,
              "endTimeUnixNano": 1694264445000000000,
              "attributes": { ... },
              "status": { "code": 0 }
            }
          ]
        }
      ]
    }
  ]
}
```

This format enables integration with any OpenTelemetry-compatible backend (Jaeger, Honeycomb, Datadog).

---

## Incident Replay Protocol

### 1. Root Cause Query

When a failure occurs, TRACER enables forensic replay using god log:

**Query Example**: "Why did compile.py fail at T+45:32?"

```sql
WITH trace_tree AS (
  SELECT * FROM god_log 
  WHERE timestamp_ms BETWEEN 1694264300000 AND 1694264400000
  AND agent_id = 'DEV_DATA'
)
SELECT * FROM trace_tree 
ORDER BY timestamp_ms DESC
```

Result: Complete causal chain from spawn to error, including all intermediate state and tool calls.

### 2. Replay Artifacts

For each query, TRACER generates:
- **Timeline**: All events in chronological order with durations
- **Call Tree**: Nested spans showing parent-child relationships
- **State Transitions**: All state changes with reasons
- **Tool Fidelity**: Which tools succeeded vs failed, with error messages
- **Context Diffs**: What context was available at each step

---

## Validation Span Criteria

When TRACER captures a validation span, it records:

**Required Attributes**:
- `validator_id`: Which agent performed validation
- `validation_type`: Type of check (schema, fidelity, completeness, etc.)
- `result`: pass/fail/warn
- `criteria_count`: How many criteria were evaluated
- `criteria_passed`: How many passed
- `criteria_failed`: List of failed criteria names
- `duration_ms`: Validation time

**Example Validation Span**:
```json
{
  "validator_id": "QA_DATA",
  "validation_type": "entity_store_schema",
  "result": "fail",
  "criteria_count": 15,
  "criteria_passed": 14,
  "criteria_failed": ["foreign_key:position_id invalid"],
  "duration_ms": 250
}
```

---

## Error Span Criteria

When any error occurs, TRACER immediately creates an error span with:

**Required Attributes**:
- `error_type`: Exception type (ValidationError, TimeoutError, etc.)
- `error_message`: Human-readable message
- `stack_trace`: Full Python traceback (if available)
- `agent_id`: Which agent failed
- `timestamp_ms`: When the error occurred
- `recovery_attempted`: Boolean (did ERROR_MONITOR spawn a DEBUGGER?)

**Example Error Span**:
```json
{
  "span_type": "error",
  "error_type": "ForeignKeyError",
  "error_message": "position_id 'pos_invalid_123' not found in entity_store",
  "stack_trace": "File compile.py line 245...",
  "agent_id": "DEV_DATA",
  "timestamp_ms": 1694264400000,
  "recovery_attempted": true
}
```

---

## Context Preservation Rules

TRACER preserves full context at each major step by capturing:

1. **Input Context**: What data was passed to this agent/tool
2. **State Snapshot**: What internal state was this when action started
3. **Output Context**: What resulted from the action
4. **Metadata**: Who approved this? When? What version?

No context is discarded during the workflow (context pruning is CONTEXT_MANAGER's responsibility, not TRACER's).

---

## Compliance Checklist

- [ ] Every agent spawn is captured with input payload hash
- [ ] Every tool call is captured with params and result
- [ ] Every state transition is logged
- [ ] Every handoff is traced with packet size and hash
- [ ] Every validation result is recorded
- [ ] Every error is captured with stack trace
- [ ] God log is append-only (no updates/deletes)
- [ ] Traces maintain parent-child relationships
- [ ] Checkpoints are created at phase boundaries
- [ ] OpenTelemetry format export available every 5 minutes
- [ ] Root cause analysis tool can query any failure within 5 minutes

---

## Integration Points

**Receives From**:
- All agents (via structured log output)
- SIMON orchestrator (phase events)

**Sends To**:
- God log storage (SQLite)
- Export system (OTLP format files)
- HANDOFF_AUDITOR (for handoff validation traces)
- ERROR_MONITOR (error span events)
- Udhay (root cause analysis queries, incident replays)

---

## Termination Conditions

TRACER terminates only after:
1. Phase 3 completes
2. All spans are written to god log
3. Final checkpoint created at T+210 min
4. OpenTelemetry export complete
5. All traces verified queryable and replay-able
