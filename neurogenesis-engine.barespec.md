SERVER: neurogenesis-engine
VERSION: 2.1
UPDATED: 2025-12-27
STATUS: Production
PORT: 3010 (UDP/InterLock), 8010 (HTTP), 9010 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: MCP Server Factory - builds, validates, and manages custom MCP servers with human-in-the-loop approval
CONFIG: /repo/neurogenesis-engine/src/constants.js

---

ARCHITECTURE

FUNCTION: Generate new MCP servers from specifications
WORKFLOW: Spec → Generate → Validate → Approve → Deploy
APPROVAL: Human-in-the-loop for all builds
QUALITY: Syntax check, MCP compliance, security scan
LEARNING: Feedback loops improve future generations

---

TOOLS (26)

## CORE NEUROGENESIS (7)

TOOL: create_synapse
INPUT: { name: string (required), purpose: string (required), tools: [{ name: string, description: string, inputSchema: object, outputSchema: object }] (required), ports?: { mcp?: number, http?: number, ws?: number }, dependencies?: string[] }
OUTPUT: { synapseId: string, name: string, status: "pending"|"generating"|"ready_for_review", estimatedGenerationTime: string }
USE: Create new MCP server from specification
EXAMPLE: create_synapse({ name: "my-server", purpose: "Handle file operations", tools: [{ name: "read_file", description: "Read a file", inputSchema: {...}, outputSchema: {...} }] })
NOTES: Starts async generation. Check get_pending_builds for completion.

TOOL: fire_neurotransmitter
INPUT: { fromServer: string (required), toServer: string (required), signal: string (required), payload?: object, priority?: "low"|"normal"|"high" }
OUTPUT: { transmitted: boolean, signalId: string, receivedAt?: string, response?: object }
USE: Trigger communication between servers
EXAMPLE: fire_neurotransmitter({ fromServer: "orchestrator", toServer: "classifier", signal: "classify_batch", payload: { files: [...] } })
NOTES: Enables server-to-server coordination.

TOOL: create_workflow
INPUT: { name: string (required), description: string (required), steps: [{ server: string, tool: string, inputMapping?: object }] (required), triggers?: string[] }
OUTPUT: { workflowId: string, name: string, stepCount: number, created: boolean }
USE: Define multi-server workflow
EXAMPLE: create_workflow({ name: "classification-pipeline", description: "Classify and organize files", steps: [{ server: "catasorter", tool: "classify_document" }, { server: "smart-file-organizer", tool: "organize_files" }] })
NOTES: Workflows can be triggered manually or by events.

TOOL: get_synaptic_network
INPUT: { includeHealth?: boolean, includeConnections?: boolean }
OUTPUT: { servers: [{ name: string, status: string, health?: object }], connections: [{ from: string, to: string, signalCount: number }], totalServers: number }
USE: View server interconnections
EXAMPLE: get_synaptic_network({ includeHealth: true, includeConnections: true })
NOTES: Network topology visualization data.

TOOL: discover_emergent_capabilities
INPUT: { servers?: string[], depth?: number (default: 2) }
OUTPUT: { emergentCapabilities: [{ combination: string[], capability: string, confidence: number, example: string }] }
USE: Find unexpected capability combinations
EXAMPLE: discover_emergent_capabilities({ servers: ["catasorter", "enterspect", "quartermaster"], depth: 3 })
NOTES: Identifies synergies between server combinations.

TOOL: get_possible_pathways
INPUT: { startServer?: string, endServer?: string, capability?: string }
OUTPUT: { pathways: [{ path: string[], capability: string, complexity: number }] }
USE: List potential server chains
EXAMPLE: get_possible_pathways({ startServer: "filesystem-test", endServer: "quartermaster" })
NOTES: Shows all valid routes through server network.

TOOL: visualize_pathway
INPUT: { pathway: string[] (required), format?: "ascii"|"mermaid"|"json" }
OUTPUT: { visualization: string, format: string }
USE: Generate pathway visualization
EXAMPLE: visualize_pathway({ pathway: ["filesystem-test", "catasorter", "quartermaster"], format: "mermaid" })
NOTES: Creates visual representation of server chain.

## META-MEMORY (5)

TOOL: log_generation_error
INPUT: { serverName: string (required), errorType: string (required), message: string (required), stack?: string, context?: object }
OUTPUT: { errorId: string, logged: boolean }
USE: Record errors during generation
EXAMPLE: log_generation_error({ serverName: "my-server", errorType: "syntax_error", message: "Unexpected token" })
NOTES: All errors tracked for pattern analysis.

TOOL: get_errors_by_server
INPUT: { serverName: string (required), limit?: number (default: 20), since?: string }
OUTPUT: { errors: [{ errorId: string, errorType: string, message: string, timestamp: string }], total: number }
USE: Errors for specific server type
EXAMPLE: get_errors_by_server({ serverName: "my-server", limit: 10 })
NOTES: Helps identify problematic server patterns.

TOOL: get_errors_by_type
INPUT: { errorType: string (required), limit?: number (default: 20), since?: string }
OUTPUT: { errors: [{ errorId: string, serverName: string, message: string, timestamp: string }], total: number }
USE: Errors by error category
EXAMPLE: get_errors_by_type({ errorType: "mcp_compliance" })
NOTES: Find systematic issues across servers.

TOOL: get_recent_errors
INPUT: { limit?: number (default: 50), severity?: "warning"|"error"|"critical" }
OUTPUT: { errors: [{ errorId: string, serverName: string, errorType: string, message: string, timestamp: string }], total: number }
USE: Latest generation errors
EXAMPLE: get_recent_errors({ limit: 20, severity: "error" })
NOTES: Quick overview of recent issues.

TOOL: get_error_stats
INPUT: { period?: "hour"|"day"|"week"|"month" (default: "week"), groupBy?: "server"|"type" }
OUTPUT: { stats: { total: number, byPeriod: object, topErrors: array }, trend: "increasing"|"decreasing"|"stable" }
USE: Error statistics and trends
EXAMPLE: get_error_stats({ period: "week", groupBy: "type" })
NOTES: Identifies improvement or regression patterns.

## QUALITY CONTROL (4)

TOOL: validate_generated_code
INPUT: { buildId: string (required), checks?: ("syntax"|"mcp"|"security"|"all")[] (default: ["all"]) }
OUTPUT: { valid: boolean, results: { syntax: object, mcp: object, security: object }, issues: array, score: number }
USE: Full validation suite
EXAMPLE: validate_generated_code({ buildId: "bld_abc123", checks: ["all"] })
NOTES: Runs all validation checks. Score 0-100.

TOOL: check_syntax_only
INPUT: { code: string (required), language?: "typescript"|"javascript" (default: "typescript") }
OUTPUT: { valid: boolean, errors: [{ line: number, column: number, message: string }], warnings: array }
USE: Quick syntax validation
EXAMPLE: check_syntax_only({ code: "const x = 1;", language: "typescript" })
NOTES: Fast check without full validation suite.

TOOL: check_mcp_compliance
INPUT: { buildId: string (required) }
OUTPUT: { compliant: boolean, issues: [{ rule: string, severity: string, message: string, location?: string }], version: string }
USE: Verify MCP protocol compliance
EXAMPLE: check_mcp_compliance({ buildId: "bld_abc123" })
NOTES: Ensures server follows MCP specification.

TOOL: scan_security_vulnerabilities
INPUT: { buildId: string (required), depth?: "quick"|"standard"|"deep" (default: "standard") }
OUTPUT: { secure: boolean, vulnerabilities: [{ severity: "low"|"medium"|"high"|"critical", type: string, description: string, location: string, recommendation: string }], score: number }
USE: Security vulnerability scan
EXAMPLE: scan_security_vulnerabilities({ buildId: "bld_abc123", depth: "deep" })
NOTES: Checks for common vulnerabilities. Score 0-100.

## DIAGNOSTIC (4)

TOOL: check_server_health
INPUT: { serverName: string (required), includeMetrics?: boolean }
OUTPUT: { healthy: boolean, status: "running"|"stopped"|"error"|"unknown", uptime?: number, lastCheck: string, metrics?: object }
USE: Health check for running servers
EXAMPLE: check_server_health({ serverName: "catasorter", includeMetrics: true })
NOTES: Checks deployed server health.

TOOL: get_performance_metrics
INPUT: { serverName: string (required), period?: "hour"|"day"|"week" (default: "day") }
OUTPUT: { metrics: { requestCount: number, avgResponseTime: number, errorRate: number, throughput: number }, breakdown: object }
USE: Server performance data
EXAMPLE: get_performance_metrics({ serverName: "catasorter", period: "day" })
NOTES: Performance over time period.

TOOL: get_health_snapshot
INPUT: { servers?: string[] }
OUTPUT: { snapshot: [{ name: string, healthy: boolean, status: string }], timestamp: string, overallHealth: "healthy"|"degraded"|"unhealthy" }
USE: Point-in-time health capture
EXAMPLE: get_health_snapshot({ servers: ["catasorter", "enterspect", "quartermaster"] })
NOTES: All servers if none specified.

TOOL: get_memory_usage
INPUT: { serverName?: string }
OUTPUT: { usage: { heapUsed: number, heapTotal: number, external: number, rss: number }, byServer?: object }
USE: Memory consumption stats
EXAMPLE: get_memory_usage({ serverName: "research-bus" })
NOTES: All servers if none specified.

## LEARNING (5)

TOOL: submit_feedback
INPUT: { buildId: string (required), rating: 1-5 (required), category: "quality"|"performance"|"usability"|"security" (required), comment: string (required), suggestion?: string }
OUTPUT: { feedbackId: string, submitted: boolean }
USE: Submit feedback on generated server
EXAMPLE: submit_feedback({ buildId: "bld_abc123", rating: 4, category: "quality", comment: "Good structure but needs better error handling" })
NOTES: Improves future generation quality.

TOOL: get_feedback_by_server
INPUT: { serverName: string (required), limit?: number (default: 20), category?: string }
OUTPUT: { feedback: [{ feedbackId: string, rating: number, category: string, comment: string, timestamp: string }], averageRating: number, total: number }
USE: Feedback for specific server
EXAMPLE: get_feedback_by_server({ serverName: "my-server", category: "quality" })
NOTES: Track feedback patterns.

TOOL: get_feedback_stats
INPUT: { period?: "week"|"month"|"all" (default: "month") }
OUTPUT: { stats: { totalFeedback: number, averageRating: number, byCategory: object, trend: string }, insights: array }
USE: Feedback statistics
EXAMPLE: get_feedback_stats({ period: "month" })
NOTES: Overall feedback health metrics.

TOOL: get_open_feedback
INPUT: { priority?: "all"|"high"|"critical" (default: "all"), limit?: number (default: 20) }
OUTPUT: { feedback: [{ feedbackId: string, buildId: string, category: string, comment: string, status: string }], total: number }
USE: Unresolved feedback items
EXAMPLE: get_open_feedback({ priority: "high" })
NOTES: Items needing attention.

TOOL: update_feedback_status
INPUT: { feedbackId: string (required), status: "open"|"in_progress"|"resolved"|"wont_fix" (required), resolution?: string }
OUTPUT: { updated: boolean, feedbackId: string, newStatus: string }
USE: Update feedback resolution
EXAMPLE: update_feedback_status({ feedbackId: "fb_123", status: "resolved", resolution: "Fixed in v2.1" })
NOTES: Track feedback lifecycle.

## CHRONOS (1)

TOOL: track_event
INPUT: { entityType: "build"|"server"|"workflow"|"error" (required), entityId: string (required), eventType: "created"|"validated"|"approved"|"rejected"|"deployed"|"error" (required), snapshot?: object }
OUTPUT: { eventId: string, timestamp: string, stored: boolean }
USE: Send event to Chronos Synapse
EXAMPLE: track_event({ entityType: "build", entityId: "bld_abc123", eventType: "approved" })
NOTES: All significant events tracked for temporal analysis.

---

KEY CONCEPTS

SYNAPSE: A generated MCP server specification
NEUROTRANSMITTER: Message/signal between servers
PATHWAY: Chain of servers for multi-step workflows
BUILD_PLAN: Generated code awaiting approval
FEEDBACK: Learning signal for improvement

---

WORKFLOW

1. SPECIFY: Define server requirements via create_synapse
2. GENERATE: Engine creates server code
3. VALIDATE: check_syntax, check_mcp_compliance, scan_security
4. REVIEW: get_build_plan shows generated code
5. APPROVE: approve_build or reject_build with feedback
6. DEPLOY: Server becomes available
7. LEARN: submit_feedback improves future generations

---

DATABASE SCHEMA

TABLE: synapses
COLUMNS: id, name, purpose, status, spec, created_at, updated_at
INDEXES: name, status, created_at

TABLE: builds
COLUMNS: id, synapse_id, status, generated_code, validation_results, approved_by, approved_at
INDEXES: synapse_id, status, approved_at

TABLE: errors
COLUMNS: id, server_name, error_type, message, stack, context, timestamp
INDEXES: server_name, error_type, timestamp

TABLE: feedback
COLUMNS: id, build_id, rating, category, comment, suggestion, status, resolution, timestamp
INDEXES: build_id, category, status, timestamp

TABLE: workflows
COLUMNS: id, name, description, steps, triggers, created_at
INDEXES: name, created_at

---

KEY FILES

SOURCE: /repo/neurogenesis-engine/
INDEX: src/index.js
TOOLS: src/tools/index.js
NEUROGENESIS: src/tools/neurogenesis.js (11 tools: 7 core + 4 approval)
META-MEMORY: src/tools/meta-memory.js
QUALITY: src/tools/quality.js
DIAGNOSTIC: src/tools/diagnostic.js
LEARNING: src/tools/learning.js
CHRONOS: src/tools/chronos.js
GENERATION: src/generation/intelligent-builder.js
DATABASE: src/database/sqlite.js
CLIENTS: src/clients/*.js (chronos, context-guardian, niws, toolee, trinity)
INTERLOCK: src/interlock/*.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
CONFIG: config/server.json, config/interlock.json, config/dspy.json
