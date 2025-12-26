TOOLS: trinity-coordinator
VERSION: 1.1
UPDATED: 2025-12-25
COUNT: 14

---

TOOL: route_task
INPUT: { task: string (required), targetMember: "desktop"|"code"|"perplexity" (required), workOrderId?: string, priority?: "low"|"normal"|"high"|"urgent" (default: "normal") }
OUTPUT: { routed_to: string, workflow: array, confidence: number }
USE: Route a task to specific Trinity member
EXAMPLE: route_task({ task: "implement authentication module", targetMember: "code" })
NOTES: Creates routing to Desktop (orchestration), Code (implementation), or Perplexity (research).

---

TOOL: execute_workflow
INPUT: { workflowName: string (required), workflowId?: string, steps?: [{ id, service, action, depends? }] }
OUTPUT: { execution_id: string, status: string, steps: array }
USE: Execute or resume a multi-step workflow
EXAMPLE: execute_workflow({ workflowName: "classify_and_organize" })

---

TOOL: delegate_to_code
INPUT: { task: string (required), filePaths?: string[], instructions?: string, workOrderId?: string, priority?: "low"|"normal"|"high"|"urgent" (default: "normal") }
OUTPUT: { delegated: boolean, work_order_id: string }
USE: Delegate implementation task to Claude Code
EXAMPLE: delegate_to_code({ task: "fix authentication bug", filePaths: ["auth.js"] })
NOTES: Creates work order in .claude-coordination/ folder.

---

TOOL: delegate_to_perplexity
INPUT: { query: string (required), domain?: string, depth?: "quick"|"thorough"|"comprehensive" (default: "thorough"), workOrderId?: string }
OUTPUT: { delegated: boolean, work_order_id: string }
USE: Delegate research task to Perplexity
EXAMPLE: delegate_to_perplexity({ query: "best practices for JWT authentication 2025", depth: "thorough" })

---

TOOL: create_workflow
INPUT: { name: string (required), description?: string, steps: [{ id, service, action, depends? }] (required) }
OUTPUT: { workflow_id: string, created: boolean }
USE: Create a reusable workflow template
EXAMPLE: create_workflow({ name: "research_then_implement", steps: [{ id: "s1", service: "perplexity", action: "research" }, { id: "s2", service: "code", action: "implement", depends: ["s1"] }] })

---

TOOL: get_routing_history
INPUT: { limit?: number (default: 50), targetMember?: "desktop"|"code"|"perplexity"|"all" (default: "all"), since?: string, workOrderId?: string }
OUTPUT: { history: array, total: number }
USE: Get history of routing decisions
EXAMPLE: get_routing_history({ limit: 20, targetMember: "code" })

---

TOOL: watch_work_orders
INPUT: { status?: "all"|"pending"|"in_progress"|"completed"|"failed" (default: "all") }
OUTPUT: { timestamp: string, totalWorkOrders: number, filtered: number, statusFilter: string, workOrders: array }
USE: Monitor .claude-coordination/ folder for work orders
EXAMPLE: watch_work_orders({ status: "pending" })

---

TOOL: get_pending_work
INPUT: {}
OUTPUT: { timestamp: string, pendingCount: number, pendingWork: [{ id: string, assignedTo: string, operation: string, priority: number, title: string, instructions: string }] }
USE: Get all pending work orders
EXAMPLE: get_pending_work({})

---

TOOL: get_work_status
INPUT: { workOrderId: string (required) }
OUTPUT: { id: string, status: string, assignedTo: string, operation: string, priority: number, createdAt: string, title: string, hasResults: boolean }
USE: Check status of a specific work order
EXAMPLE: get_work_status({ workOrderId: "wo_123" })

---

TOOL: get_trinity_status
INPUT: {}
OUTPUT: { timestamp: string, trinityStatus: "idle"|"active"|"busy", summary: { total: number, pending: number, inProgress: number, completed: number, failed: number }, alerts?: array }
USE: Get overall Trinity coordination status
EXAMPLE: get_trinity_status({})

---

TOOL: notify_desktop
INPUT: { message: string (required), workOrderId?: string }
OUTPUT: { notified: boolean, message: string, workOrderId: string, timestamp: string }
USE: Send notification to Desktop
EXAMPLE: notify_desktop({ message: "Task completed", workOrderId: "wo_123" })

---

TOOL: get_pending_handoffs
INPUT: { member?: "code"|"desktop"|"perplexity" (default: "desktop") }
OUTPUT: { timestamp: string, member: string, totalHandoffs: number, myTurn: number, handoffs: array }
USE: Get handoffs waiting for a Trinity member
EXAMPLE: get_pending_handoffs({ member: "code" })

---

TOOL: analyze_work_order_complexity
INPUT: { workOrderId: string (required) }
OUTPUT: { workOrderId: string, analysis: object, decision: object, recommendation: string }
USE: Analyze work order complexity and get reasoning model recommendation
EXAMPLE: analyze_work_order_complexity({ workOrderId: "WO-2025-12-25-001" })
NOTES: Returns complexity score and whether reasoning model is needed.

---

TOOL: suggest_model_provider
INPUT: { workOrderId: string (required) }
OUTPUT: { workOrderId: string, routing: object, costValid: boolean, recommendation: string, rationale: string, estimatedCost: string, confidence: string }
USE: Get optimal model provider suggestion for a work order
EXAMPLE: suggest_model_provider({ workOrderId: "WO-2025-12-25-001" })
NOTES: Routes to Anthropic (Claude), OpenAI (GPT), Google (Gemini), or Perplexity based on task requirements.

---

TRINITY MEMBERS:
- Desktop: Orchestration, user interaction, approval
- Code: Implementation, file operations, scripting
- Perplexity: Research, web search, fact-finding

COORDINATION: .claude-coordination/ folder
HANDOFFS: File-based protocol between members
