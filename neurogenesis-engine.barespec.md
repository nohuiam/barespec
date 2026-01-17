SERVER: neurogenesis-engine
VERSION: 2.1
UPDATED: 2026-01-17
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

TOOL CATEGORIES (30 total)

CORE_NEUROGENESIS (11): create_synapse, fire_neurotransmitter, create_workflow, get_synaptic_network, discover_emergent_capabilities, get_possible_pathways, visualize_pathway, get_pending_builds, get_build_plan, approve_build, reject_build

BUILD_APPROVAL (subset of CORE_NEUROGENESIS):
- get_pending_builds: List all builds awaiting human approval
- get_build_plan: Get detailed build plan for a pending build
- approve_build: Approve pending build to continue generation
- reject_build: Reject a pending build with reason

META_MEMORY (5):
- log_generation_error: Record errors during generation
- get_errors_by_server: Errors for specific server type
- get_errors_by_type: Errors by error category
- get_recent_errors: Latest generation errors
- get_error_stats: Error statistics and trends

QUALITY_CONTROL (4):
- validate_generated_code: Full validation suite
- check_syntax_only: Quick syntax validation
- check_mcp_compliance: Verify MCP protocol compliance
- scan_security_vulnerabilities: Security vulnerability scan

DIAGNOSTIC (4):
- check_server_health: Health check for running servers
- get_performance_metrics: Server performance data
- get_health_snapshot: Point-in-time health capture
- get_memory_usage: Memory consumption stats

LEARNING (5):
- submit_feedback: Submit feedback on generated server
- get_feedback_by_server: Feedback for specific server
- get_feedback_stats: Feedback statistics
- get_open_feedback: Unresolved feedback items
- update_feedback_status: Update feedback resolution

CHRONOS (1):
- track_event: Send event to Chronos Synapse

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

INTEGRATIONS

CHRONOS_SYNAPSE: Event tracking via track_event
INTERLOCK: UDP mesh at port 3013
SECURITY: All generated code scanned before approval

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

---

LAYERS

1. MCP stdio (30 tools) - Primary interface
2. InterLock UDP mesh (port 3010) - Peer communication
3. HTTP REST API (port 8010) - External integrations
4. WebSocket real-time (port 9010) - Live updates

---

HTTP REST API (port 8010)

ENDPOINT: GET /health
OUTPUT: { status, server, version, uptime }
USE: Health check

ENDPOINT: GET /api/tools
OUTPUT: { tools: [{ name, description, inputSchema }], count: number }
USE: List all MCP tools for bop-gateway integration

ENDPOINT: POST /api/tools/:toolName
INPUT: { arguments: object } or direct args object
OUTPUT: { success: boolean, result: object } or { success: false, error: string }
USE: Execute MCP tool via HTTP for bop-gateway integration

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
