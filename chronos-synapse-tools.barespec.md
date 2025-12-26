TOOLS: chronos-synapse
VERSION: 2.0
UPDATED: 2025-12-25
COUNT: 19

---

## PHASE 1: EVENT TRACKING (✅ Implemented)

---

TOOL: track_event
STATUS: ✅ Implemented
INPUT: { entityType: "project"|"file"|"server"|"error"|"process"|"metric"|"ecosystem" (required), entityId: string (required), eventType: "created"|"modified"|"deleted"|"error"|"recovered"|"validated"|"deployed" (required), snapshot?: object, causedBy?: string, correlationId?: string, tags?: string[] }
OUTPUT: { eventId: string, timestamp: string, stored: boolean, causalChainLength?: number }
USE: Record single event with optional causal linking and correlation grouping
EXAMPLE: track_event({ entityType: "file", entityId: "auth.ts", eventType: "modified", causedBy: "evt_abc123" })
NOTES: <0.08ms per event. causedBy creates causal chain link.

---

TOOL: track_snapshot
STATUS: ✅ Implemented
INPUT: { entityType: string (required), entityId: string (required), label?: string, state: object (required) }
OUTPUT: { snapshotId: string, timestamp: string, canCompareWith: string[] }
USE: Capture complete entity state at point in time
EXAMPLE: track_snapshot({ entityType: "server", entityId: "api-server", label: "before-migration", state: { version: "1.0", config: {...} } })
NOTES: <2ms per snapshot. Label for human-readable identification.

---

TOOL: batch_track_events
STATUS: ✅ Implemented
INPUT: { correlationId: string (required), events: [{ entityType: string, entityId: string, eventType: string, snapshot?: object }] (required) }
OUTPUT: { correlationId: string, eventsRecorded: number, eventIds: string[], causalLinksCreated: number }
USE: Record multiple events atomically with shared correlation and auto-sequential linking
EXAMPLE: batch_track_events({ correlationId: "deploy-v2", events: [{ entityType: "server", entityId: "api", eventType: "modified" }, { entityType: "file", entityId: "config.json", eventType: "modified" }] })
NOTES: <5ms for 4 events. All-or-nothing transaction. Auto-creates causal links between events.

---

TOOL: get_timeline
STATUS: ✅ Implemented
INPUT: { entityType: string (required), entityId: string (required), startTime?: string, endTime?: string, eventTypes?: string[], limit?: number (default: 100) }
OUTPUT: { entity: string, timelineSpan: string, eventCount: number, events: [{ timestamp: string, eventType: string, summary: string, causedBy?: string, snapshot?: object }] }
USE: Query event history for entity with time range and type filters
EXAMPLE: get_timeline({ entityType: "server", entityId: "api-server", eventTypes: ["error", "recovered"], limit: 20 })
NOTES: <10ms for 10 events. Returns human-readable summary per event.

---

## PHASE 2: CAUSALITY ANALYSIS (✅ Implemented)

---

TOOL: analyze_causality
STATUS: ✅ Implemented
INPUT: { effectEventId: string (required), maxDepth?: number (default: 5), confidenceThreshold?: number (default: 0.6), timeWindowHours?: number (default: 24), includeCorrelations?: boolean }
OUTPUT: { causalChains: [{ path: string[], confidence: number, evidence: string[], explanation: string }], rootCauses: string[], alternativeExplanations?: string[] }
USE: Root cause analysis - trace backwards from effect to find causes
EXAMPLE: analyze_causality({ effectEventId: "evt_error123", maxDepth: 3 })
NOTES: Follows causedBy links and correlations to find root causes.

---

TOOL: detect_correlation
STATUS: ✅ Implemented
INPUT: { eventId?: string, eventType?: string, timeWindowHours?: number (default: 168), minCorrelationScore?: number (default: 0.5), maxResults?: number (default: 20) }
OUTPUT: { correlations: [{ eventA: string, eventB: string, score: number, occurrences: number, pattern: string }] }
USE: Find events that frequently occur together (may not be causal)
EXAMPLE: detect_correlation({ eventType: "deploy", timeWindowHours: 720 })
NOTES: Statistical correlation, not causation.

---

TOOL: trace_impact
STATUS: ✅ Implemented
INPUT: { causeEventId: string (required), maxDepth?: number (default: 5), includeIndirect?: boolean }
OUTPUT: { directImpacts: string[], indirectImpacts?: string[], cascadeDepth: number, affectedEntities: string[], impactGraph: object }
USE: Forward impact analysis - what did this event cause downstream
EXAMPLE: trace_impact({ causeEventId: "evt_config_change", maxDepth: 3, includeIndirect: true })
NOTES: Follows causal chains forward to map impact.

---

TOOL: explain_change
STATUS: ✅ Implemented
INPUT: { entityType: string (required), entityId: string (required), change: string (required), when?: string }
OUTPUT: { explanation: string, context: string[], probableCause: string, confidence: number, severity: string, recommendations: string[] }
USE: Natural language explanation of why something changed
EXAMPLE: explain_change({ entityType: "metric", entityId: "response_time", change: "increased 50%" })
NOTES: Synthesizes timeline, correlations, and causal chains into explanation.

---

## PHASE 3: PATTERN RECOGNITION (✅ Implemented)

---

TOOL: detect_cycles
STATUS: ✅ Implemented
INPUT: { entityType: string (required), entityId: string (required), minOccurrences?: number (default: 3), searchBack?: string (default: "30 days") }
OUTPUT: { cycleDetected: boolean, pattern?: { type: "weekly"|"daily"|"monthly"|"custom", description: string, occurrences: number, confidence: number, nextExpected: string }, hypothesis: string, recommendation: string }
USE: Detect recurring temporal patterns in events
EXAMPLE: detect_cycles({ entityType: "error", entityId: "timeout_error", minOccurrences: 5 })
NOTES: Identifies daily, weekly, monthly, or custom cycles.

---

TOOL: detect_trends
STATUS: ✅ Implemented
INPUT: { metric: string (required), entityType: string (required), entityId: string (required), period?: string (default: "30 days") }
OUTPUT: { trend: "increasing"|"decreasing"|"stable", direction: "upward"|"downward"|"flat", velocity: string, confidence: number, projection?: { in7Days: number, in30Days: number }, interpretation: string, recommendation: string }
USE: Detect trends in metrics over time
EXAMPLE: detect_trends({ metric: "error_rate", entityType: "server", entityId: "api-server" })
NOTES: Linear regression on metric values over time.

---

TOOL: detect_anomalies
STATUS: ✅ Implemented
INPUT: { entityType: string (required), entityId: string (required), metric: string (required), sensitivity?: "low"|"medium"|"high", baselineDays?: number (default: 7) }
OUTPUT: { anomaliesDetected: number, anomalies: [{ timestamp: string, expected: string, actual: string, deviation: string, severity: string, possibleCauses?: string[] }], recommendation?: string }
USE: Detect events or values that break normal patterns
EXAMPLE: detect_anomalies({ entityType: "server", entityId: "api-server", metric: "response_time", sensitivity: "medium" })
NOTES: Statistical outlier detection against baseline.

---

## PHASE 4: PREDICTION (✅ Implemented)

---

TOOL: predict_next_event
STATUS: ✅ Implemented
INPUT: { eventType: string (required), entityType: string (required), entityId: string (required), basedOnHistory?: string (default: "30 days"), confidence?: "low"|"medium"|"high" }
OUTPUT: { predictedTime: string, confidence: number, reasoning: string[], basedOn: { eventCount: number, patternType: string }, preventiveActions?: string[] }
USE: Predict when next occurrence of event will happen
EXAMPLE: predict_next_event({ eventType: "error", entityType: "server", entityId: "api-server" })
NOTES: Uses cycle detection and trend analysis.

---

TOOL: predict_bottleneck
STATUS: ✅ Implemented
INPUT: { scope: "ecosystem"|"server"|"process" (required), horizon?: string (default: "30 days"), metric?: string }
OUTPUT: { bottlenecks: [{ component: string, metric: string, currentState: string, projectedState: string, capacity: string, timeToBottleneck: string, confidence: number, recommendation: string }] }
USE: Predict future system bottlenecks before they occur
EXAMPLE: predict_bottleneck({ scope: "ecosystem", horizon: "14 days" })
NOTES: Analyzes growth trends against known capacity limits.

---

TOOL: forecast_metric
STATUS: ✅ Implemented
INPUT: { metric: string (required), entityType: string (required), forecastDays?: number (default: 7), includeConfidenceInterval?: boolean }
OUTPUT: { forecast: [{ date: string, predicted: number, confidenceLow?: number, confidenceHigh?: number }], trend: string, confidence: number, methodology: string }
USE: Forecast future metric values with confidence intervals
EXAMPLE: forecast_metric({ metric: "daily_requests", entityType: "server", forecastDays: 14, includeConfidenceInterval: true })
NOTES: Time series forecasting with confidence bands.

---

TOOL: what_if_analysis
STATUS: ✅ Implemented
INPUT: { scenario: string (required), affectedMetrics: string[] (required), basedOnHistory?: string }
OUTPUT: { projectedImpact: { [metric: string]: { current: string, projected: string, change: string } }, recommendation: string, confidence: number, assumptions: string[] }
USE: Simulate impact of hypothetical changes
EXAMPLE: what_if_analysis({ scenario: "Add 5 more API servers", affectedMetrics: ["response_time", "throughput"] })
NOTES: Uses historical correlations to project impact.

---

## PHASE 5: OPTIMIZATION (✅ Implemented)

---

TOOL: optimize_timing
STATUS: ✅ Implemented
INPUT: { action: string (required), constraints: { duration?: string, requireLowLoad?: boolean, mustCompleteBy?: string }, optimizeFor?: "minimal_impact"|"fastest_execution"|"lowest_cost" }
OUTPUT: { optimalWindow: { start: string, end: string, reasoning?: string[], score?: number }, alternativeWindows?: [{ start: string, end: string, score: number }], cronExpression?: string }
USE: Find optimal time to perform an action based on system patterns
EXAMPLE: optimize_timing({ action: "database migration", constraints: { duration: "2 hours", requireLowLoad: true }, optimizeFor: "minimal_impact" })
NOTES: Analyzes historical load patterns to find best windows.

---

TOOL: suggest_schedule
STATUS: ✅ Implemented
INPUT: { task: string (required), frequency: "daily"|"weekly"|"monthly" (required), duration?: string, priority: "low"|"medium"|"high" }
OUTPUT: { recommendedSchedule: { frequency: string, dayOfWeek?: string, time: string, reasoning: string }, cronExpression: string, nextRun: string }
USE: Suggest optimal recurring schedule for a task
EXAMPLE: suggest_schedule({ task: "backup", frequency: "daily", priority: "high" })
NOTES: Considers system load patterns and task priority.

---

TOOL: optimize_sequence
STATUS: ✅ Implemented
INPUT: { operations: [{ id: string, duration: string, dependencies: string[] }] (required), goal: "minimize_total_time"|"maximize_reliability" }
OUTPUT: { optimizedSequence: [{ operation: string, startTime: string, endTime: string, parallel?: boolean }], totalTime: string, savings?: string, criticalPath: string[] }
USE: Optimize order of operations based on dependencies and timing
EXAMPLE: optimize_sequence({ operations: [{ id: "migrate", duration: "1h", dependencies: [] }, { id: "test", duration: "30m", dependencies: ["migrate"] }], goal: "minimize_total_time" })
NOTES: Critical path analysis with parallel execution where possible.

---

TOOL: find_maintenance_window
STATUS: ✅ Implemented
INPUT: { maintenanceDuration: string (required), allowableDowntime?: string, criticalServices?: string[], within?: string (default: "next 7 days") }
OUTPUT: { recommendedWindow: { start: string, end: string, impact: string, reasoning: string[] }, riskAssessment: string, rollbackPlan?: string, alternativeWindows?: [{ start: string, end: string, impact: string }] }
USE: Find best time for system maintenance with minimal impact
EXAMPLE: find_maintenance_window({ maintenanceDuration: "4 hours", criticalServices: ["api-server", "auth-server"], within: "next 14 days" })
NOTES: Analyzes usage patterns, SLAs, and service criticality.

---

## DATABASE SCHEMA

TABLE: events (append-only, immutable)
COLUMNS: id, timestamp, entity_type, entity_id, event_type, snapshot, caused_by, correlation_id, source, confidence, tags
INDEXES: timestamp, entity, type, correlation, source, caused_by

TABLE: snapshots
COLUMNS: id, entity_type, entity_id, label, snapshot_at, state, event_count
AUTO_CREATE: Every 6 hours OR 5000 events

TABLE: causal_chains
COLUMNS: id, cause_event_id, effect_event_id, confidence, reasoning
INDEXES: cause, effect

TABLE: patterns
COLUMNS: id, pattern_type, entity_type, entity_id, description, confidence, first_seen, last_seen, occurrences, metadata, active
INDEXES: active_type, entity, pattern_type

TABLE: predictions
COLUMNS: id, prediction_type, entity_type, entity_id, predicted_event, predicted_time, confidence, reasoning, based_on_events, actual_outcome, accuracy
INDEXES: time, entity, type, accuracy
