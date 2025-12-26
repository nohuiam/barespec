SERVER: trinity-coordinator
VERSION: 1.0
UPDATED: 2025-12-26
STATUS: Production
PORT: 3012 (UDP/InterLock), 8012 (HTTP), 9012 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: Multi-Claude orchestration for Desktop, Code, and Perplexity coordination
CONFIG: ENV TRINITY_HTTP_PORT, TRINITY_INTERLOCK_PORT, TRINITY_WEBSOCKET_PORT

---

ARCHITECTURE

MEMBERS: Desktop (orchestration), Code (implementation), Perplexity (research)
COORDINATION: .claude-coordination/ folder for work orders
HANDOFFS: File-based handoff protocol between Trinity members

TOOL: route_task
INPUT: { task: string (required), targetMember: "desktop"|"code"|"perplexity" (required), workOrderId?: string, priority?: "low"|"normal"|"high"|"urgent" (default: "normal") }
OUTPUT: { routed_to, workflow: [], confidence }
USE: Route a task to specific Trinity member
EXAMPLE: route_task({ task: "implement authentication module", targetMember: "code" })

TOOL: execute_workflow
INPUT: { workflowName: string (required), workflowId?: string, steps?: [{ id, service, action, depends? }] }
OUTPUT: { execution_id, status, steps: [] }
USE: Execute or resume a multi-step workflow
EXAMPLE: execute_workflow({ workflowName: "classify_and_organize" })

TOOL: delegate_to_code
INPUT: { task: string (required), filePaths?: string[], instructions?: string, workOrderId?: string, priority?: "low"|"normal"|"high"|"urgent" (default: "normal") }
OUTPUT: { delegated: boolean, work_order_id }
USE: Delegate implementation task to Claude Code
EXAMPLE: delegate_to_code({ task: "fix authentication bug", filePaths: ["auth.js"] })

TOOL: delegate_to_perplexity
INPUT: { query: string (required), domain?: string, depth?: "quick"|"thorough"|"comprehensive" (default: "thorough"), workOrderId?: string }
OUTPUT: { delegated: boolean, work_order_id }
USE: Delegate research task to Perplexity
EXAMPLE: delegate_to_perplexity({ query: "best practices for JWT authentication 2025", depth: "thorough" })

TOOL: create_workflow
INPUT: { name: string (required), description?: string, steps: [{ id, service, action, depends? }] (required) }
OUTPUT: { workflow_id, created: boolean }
USE: Create a reusable workflow template
EXAMPLE: create_workflow({ name: "research_then_implement", steps: [{ id: "s1", service: "perplexity", action: "research" }] })

TOOL: get_routing_history
INPUT: { limit?: number (default: 50), targetMember?: "desktop"|"code"|"perplexity"|"all" (default: "all"), since?: string, workOrderId?: string }
OUTPUT: { history: [], total }
USE: Get history of routing decisions
EXAMPLE: get_routing_history({ limit: 20, targetMember: "code" })

TOOL: watch_work_orders
INPUT: { status?: "all"|"pending"|"in_progress"|"completed"|"failed" (default: "all") }
OUTPUT: { timestamp, totalWorkOrders, filtered, statusFilter, workOrders: [] }
USE: Monitor .claude-coordination/ folder for work orders
EXAMPLE: watch_work_orders({ status: "pending" })

TOOL: get_pending_work
INPUT: {}
OUTPUT: { timestamp, pendingCount, pendingWork: [{ id, assignedTo, operation, priority, title, instructions }] }
USE: Get all work orders with status "pending" that need execution
EXAMPLE: get_pending_work({})

TOOL: get_work_status
INPUT: { workOrderId: string (required) }
OUTPUT: { id, status, assignedTo, operation, priority, createdAt, title, hasResults }
USE: Check status of a specific work order by ID
EXAMPLE: get_work_status({ workOrderId: "wo_123" })

TOOL: get_trinity_status
INPUT: {}
OUTPUT: { timestamp, trinityStatus: "idle"|"active"|"busy", summary: { total, pending, inProgress, completed, failed }, alerts?: [] }
USE: Get overall Trinity coordination status
EXAMPLE: get_trinity_status({})

TOOL: notify_desktop
INPUT: { message: string (required), workOrderId?: string }
OUTPUT: { notified: boolean, message, workOrderId, timestamp }
USE: Send notification message to Desktop (visible in logs)
EXAMPLE: notify_desktop({ message: "Task completed", workOrderId: "wo_123" })

TOOL: get_pending_handoffs
INPUT: { member?: "code"|"desktop"|"perplexity" (default: "desktop") }
OUTPUT: { timestamp, member, totalHandoffs, myTurn, handoffs: [] }
USE: Get handoffs waiting for current Trinity member's contribution
EXAMPLE: get_pending_handoffs({ member: "code" })

---

TRINITY MEMBERS

- Desktop: Orchestration, planning, user interaction
- Code: Implementation, debugging, testing
- Perplexity: Research, fact-checking, external knowledge

---

KEY FILES

SOURCE: /repo/trinitycoordinator/
INDEX: src/index.ts (or .js)
COORDINATION: .claude-coordination/ (work orders folder)
