TOOLS: intelligent-router
VERSION: 1.0
UPDATED: 2025-12-25
COUNT: 2

---

TOOL: route_request
INPUT: { request: string (required), context?: { currentPath?: string, recentOperations?: string[], timeConstraints?: string, userPreferences?: object }, routing?: { autoExecute?: boolean, preferredTools?: string[], maxComplexity?: 1-5 }, learning?: { trackPerformance?: boolean, savePattern?: boolean } }
OUTPUT: { intent: string, confidence: number, workflow: { steps: [{ tool: string, params: object, expected_output: string }], estimated_duration: number }, alternativeWorkflows: array, routingDecision: object }
USE: Route natural language request to appropriate tools with optimal workflow
EXAMPLE: route_request({ request: "organize my inbox safely", routing: { autoExecute: false } })
NOTES: autoExecute=true runs workflow if confidence > 0.85. maxComplexity limits workflow depth.

---

TOOL: get_routing_history
INPUT: { limit?: number (default: 10), offset?: number (default: 0), intent?: string, status?: string }
OUTPUT: { history: [{ request: string, intent: string, workflow: object, execution_time: number, status: string }], total: number, patterns: array }
USE: View past routing decisions and patterns
EXAMPLE: get_routing_history({ limit: 20, status: "completed" })
NOTES: patterns shows frequently used request→workflow mappings.

---

INTENTS:
- file_organization: Move, rename, organize files
- classification: GLEC classify documents
- search: Find files by content or metadata
- analysis: Analyze documents or directories
- batch_processing: Process multiple files
- maintenance: Index refresh, cleanup, health checks

COMPLEXITY LEVELS:
- 1: Single tool, direct execution
- 2: Two tools, sequential
- 3: Multiple tools, some parallel
- 4: Complex workflow, conditionals
- 5: Multi-phase, requires validation

LEARNING: Tracks successful patterns for future routing optimization
