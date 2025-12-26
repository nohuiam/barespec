SERVER: chronos-synapse
VERSION: 2.0
UPDATED: 2025-12-26
STATUS: Production (All 5 phases implemented)
PORT: 8011 (HTTP only)
MCP: stdio transport (stdin/stdout JSON-RPC)
UDP: None (no InterLock mesh)
WS: None (not implemented)
PURPOSE: Temporal intelligence - event tracking, causal analysis, pattern detection, prediction, optimization
CONFIG: ENV HTTP_PORT (default 8011)

---

ARCHITECTURE

DATABASE: SQLite (/chronos.db)
TABLES: events, snapshots, causal_chains, patterns, predictions
PERFORMANCE: <0.08ms per event, 41 bytes per event compressed

---

PHASE OVERVIEW

PHASE_1: Event Tracking
STATUS: ✅ Implemented
TOOLS: track_event, track_snapshot, batch_track_events, get_timeline

PHASE_2: Causality Analysis
STATUS: ✅ Implemented
TOOLS: analyze_causality, detect_correlation, trace_impact, explain_change

PHASE_3: Pattern Recognition
STATUS: ✅ Implemented
TOOLS: detect_cycles, detect_trends, detect_anomalies

PHASE_4: Prediction
STATUS: ✅ Implemented
TOOLS: predict_next_event, predict_bottleneck, forecast_metric, what_if_analysis

PHASE_5: Optimization
STATUS: ✅ Implemented
TOOLS: optimize_timing, suggest_schedule, optimize_sequence, find_maintenance_window

---

TOOL SUMMARY (19 total)

EVENT_TRACKING (4):
- track_event: Record single event with optional causal linking
- track_snapshot: Capture complete entity state at point in time
- batch_track_events: Record multiple events atomically with auto-linking
- get_timeline: Query event history for entity with filters

CAUSALITY (4):
- analyze_causality: Root cause analysis, trace backwards from effect
- detect_correlation: Find events that occur together
- trace_impact: Forward impact analysis, what did event cause
- explain_change: Natural language explanation of why something changed

PATTERNS (3):
- detect_cycles: Detect recurring temporal patterns (daily/weekly/monthly)
- detect_trends: Detect metric trends (increasing/decreasing/stable)
- detect_anomalies: Detect events that break normal patterns

PREDICTION (4):
- predict_next_event: When will next occurrence happen
- predict_bottleneck: Predict future system bottlenecks
- forecast_metric: Forecast metric value with confidence intervals
- what_if_analysis: Simulate impact of hypothetical changes

OPTIMIZATION (4):
- optimize_timing: Find optimal time for action
- suggest_schedule: Suggest optimal recurring schedule
- optimize_sequence: Optimize order of operations
- find_maintenance_window: Find best time for maintenance

---

ENTITY TYPES: project, file, server, error, process, metric, ecosystem
EVENT TYPES: created, modified, deleted, error, recovered, validated, deployed

---

INTEGRATIONS

LANGFUSE: Event forwarding for monitoring
INTERLOCK: UDP mesh at port 3011
CHRONOS_CLIENTS: All servers can track events here

---

KEY FILES

SOURCE: /repo/Chronos_Synapse/
INDEX: src/index.ts
TYPES: src/types/chronos-types.ts
TOOLS_P1: src/tools/event-tracking.ts
TOOLS_P2: src/tools/causality-analysis.ts
TOOLS_P3: src/tools/pattern-recognition.ts
TOOLS_P4: src/tools/predictions.ts
TOOLS_P5: src/tools/optimization.ts
DATABASE: src/database/event-store.ts
MONITORING: src/monitoring/langfuse.ts
