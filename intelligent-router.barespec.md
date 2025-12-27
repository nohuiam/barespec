SERVER: intelligent-router
VERSION: 1.1
UPDATED: 2025-12-26
STATUS: Production
PORT: 3020 (UDP/InterLock), 8020 (HTTP), 9020 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Claude API (intent classification)
PURPOSE: Natural language to automated multi-tool workflow routing
CONFIG: /repo/intelligentrouter/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
FUNCTION: Intent classification and workflow orchestration
LEARNING: Pattern recognition from successful routings
COMPLEXITY: 1-5 scale (1=single tool, 5=complex multi-step)
INTERLOCK: BaNano protocol, signals 0xC0-0xCF (ROUTE_REQUEST, ROUTE_RESPONSE, ROUTE_COMPLETE)

---

TOOLS (2)

TOOL: route_request
INPUT: { request: string (required), context?: { currentPath?: string, recentOperations?: string[], timeConstraints?: string, userPreferences?: object }, routing?: { autoExecute?: boolean, preferredTools?: string[], maxComplexity?: 1-5 }, learning?: { trackPerformance?: boolean, savePattern?: boolean } }
OUTPUT: { intent: string, confidence: number, workflow: { steps: [{ tool: string, params: object, expected_output: string }], estimated_duration: number }, alternativeWorkflows: array, routingDecision: object }
USE: Route natural language request to appropriate tools with optimal workflow
EXAMPLE: route_request({ request: "organize my inbox safely", routing: { autoExecute: false } })
NOTES: autoExecute=true runs workflow if confidence > 0.85. maxComplexity limits workflow depth.

TOOL: get_routing_history
INPUT: { limit?: number (default: 10), offset?: number (default: 0), intent?: string, status?: string }
OUTPUT: { history: [{ request: string, intent: string, workflow: object, execution_time: number, status: string }], total: number, patterns: array }
USE: View past routing decisions and patterns for learning and debugging
EXAMPLE: get_routing_history({ limit: 20, status: "completed" })
NOTES: patterns shows frequently used request→workflow mappings.

---

INTENTS

- file_organization: Organize, sort, move files
- classification: GLEC categorization
- search: Find files, content, patterns
- analysis: Analyze documents, detect issues
- batch_processing: Bulk operations
- maintenance: Cleanup, deduplication, archival

---

COMPLEXITY LEVELS

- 1: Single tool, direct execution
- 2: Two tools, sequential
- 3: Multiple tools, some parallel
- 4: Complex workflow, conditionals
- 5: Multi-phase, requires validation

LEARNING: Tracks successful patterns for future routing optimization

---

KEY FILES

SOURCE: /repo/intelligentrouter/
INDEX: src/index.ts
ROUTER: src/services/intelligentRouter.ts

DEPENDENCIES: Claude API (intent classification), Zod (schema validation)
