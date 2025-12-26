SERVER: tool-registry
VERSION: 2.0
UPDATED: 2025-12-26
STATUS: Production
PORT: 3004 (UDP/InterLock), 8004 (HTTP), 9004 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: SQLite, Langfuse, Redis (optional)
PURPOSE: Meta-infrastructure AstroSentry for tool discovery, capability indexing, and ecosystem coordination
CONFIG: /repo/Toolee/Tool_Registry/config/interlock.json

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3) - 15 tables (tools, approvals, ratings, curation, patterns, deprecations, compatibility, notifications, learning, analytics, servers)
WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
OBSERVABILITY: Langfuse integration for LLM observability
DISCOVERY: Automatic tool discovery from connected servers

---

TOOLS (65)

## CORE REGISTRY (9 tools)

TOOL: tool_registry_add
INPUT: { toolName: string (required), serverName: string (required), description: string (required), riskLevel: "low"|"medium"|"high"|"critical" (required), category: string (required), requiresApproval?: boolean, tags?: string[] }
OUTPUT: { success: boolean, toolId: uuid, message: string, backupCreated: boolean, approvalRequired?: boolean, requestId?: uuid }
USE: Register a new tool in the registry
EXAMPLE: tool_registry_add({ toolName: "my_tool", serverName: "my-server", description: "Does something", riskLevel: "low", category: "utility" })

TOOL: tool_registry_update
INPUT: { toolName: string (required), description?: string, riskLevel?: "low"|"medium"|"high"|"critical", category?: string, requiresApproval?: boolean, enabled?: boolean }
OUTPUT: { success: boolean, message: string }
USE: Update an existing tool's metadata
EXAMPLE: tool_registry_update({ toolName: "my_tool", enabled: false })

TOOL: tool_registry_delete
INPUT: { toolName: string (required) }
OUTPUT: { success: boolean, message: string, approvalRequired: boolean, requestId: uuid }
USE: Delete a tool from the registry (may require approval)
EXAMPLE: tool_registry_delete({ toolName: "deprecated_tool" })

TOOL: tool_registry_get
INPUT: { toolName: string (required) }
OUTPUT: { toolName, serverName, description, riskLevel, category, requiresApproval, enabled, tags, usageCount, successRate?, lastUsed?, discoveredAt }
USE: Get full details of a specific tool
EXAMPLE: tool_registry_get({ toolName: "quartermaster_search" })

TOOL: tool_registry_list
INPUT: { enabled?: boolean, serverName?: string, category?: string, riskLevel?: "low"|"medium"|"high"|"critical", requiresApproval?: boolean, tags?: string[] }
OUTPUT: { tools: [{ toolName, serverName, description, riskLevel, category, enabled, healthTag, usageCount }] }
USE: List tools with optional filtering
EXAMPLE: tool_registry_list({ category: "search", enabled: true })

TOOL: tool_registry_stats
INPUT: {}
OUTPUT: { totalTools, enabledTools, disabledTools, byCategory: {}, byServer: {}, byRiskLevel: {}, totalUsage?, averageSuccessRate?, topUsedTools: [] }
USE: Get comprehensive registry statistics
EXAMPLE: tool_registry_stats({})

TOOL: tool_registry_approve
INPUT: { requestId: string (required), reason?: string }
OUTPUT: { success: boolean, message: string }
USE: Approve a pending operation request
EXAMPLE: tool_registry_approve({ requestId: "abc-123", reason: "Reviewed and safe" })

TOOL: tool_registry_deny
INPUT: { requestId: string (required), reason: string (required) }
OUTPUT: { success: boolean, message: string }
USE: Deny a pending operation request
EXAMPLE: tool_registry_deny({ requestId: "abc-123", reason: "Risk too high" })

TOOL: tool_registry_pending
INPUT: {}
OUTPUT: { pending: [{ requestId, operationType, requestedBy, requestedAt, expiresAt, toolData }] }
USE: List all pending approval requests
EXAMPLE: tool_registry_pending({})

## CAPABILITY MATCHING (1 tool) - MOST IMPORTANT

TOOL: tool_registry_can_do
INPUT: { query: string (required) }
OUTPUT: { canDo: boolean, matchingCapabilities: [{ capability, tools: [] }], explanation: string, suggestions: [] }
USE: Natural language query - "Can I do X?" - routes to matching tools
EXAMPLE: tool_registry_can_do({ query: "search for files by content" })

## CURATION (4 tools)

TOOL: tool_curator_flag
INPUT: { toolName: string (required), flag: "spam"|"malicious"|"deprecated"|"broken"|"duplicate"|"low-quality" (required), reason: string (required), curator: string (required) }
OUTPUT: { success: boolean, message: string }
USE: Flag a tool for quality issues
EXAMPLE: tool_curator_flag({ toolName: "bad_tool", flag: "broken", reason: "Always errors", curator: "admin" })

TOOL: tool_curator_verify
INPUT: { toolName: string (required), curator: string (required), notes?: string }
OUTPUT: { success: boolean, message: string }
USE: Mark a tool as verified/curated
EXAMPLE: tool_curator_verify({ toolName: "good_tool", curator: "admin" })

TOOL: tool_curator_add_note
INPUT: { toolName: string (required), note: string (required), curator: string (required) }
OUTPUT: { success: boolean, message: string }
USE: Add a curation note to a tool
EXAMPLE: tool_curator_add_note({ toolName: "complex_tool", note: "Requires Redis", curator: "admin" })

TOOL: tool_curator_get_flagged
INPUT: { flag?: "spam"|"malicious"|"deprecated"|"broken"|"duplicate"|"low-quality" }
OUTPUT: { flagged: [{ flagId, toolId, toolName, serverName, flag, reason, curator, createdAt, resolved }] }
USE: Get all flagged tools
EXAMPLE: tool_curator_get_flagged({ flag: "broken" })

## RATING (3 tools)

TOOL: tool_rating_rate
INPUT: { toolName: string (required), rating: number 1-5 (required), userId: string (required) }
OUTPUT: { success: boolean, message: string }
USE: Rate a tool 1-5 stars
EXAMPLE: tool_rating_rate({ toolName: "quartermaster_search", rating: 5, userId: "user123" })

TOOL: tool_rating_review
INPUT: { toolName: string (required), review: string (required), rating: number 1-5 (required), userId: string (required) }
OUTPUT: { success: boolean, message: string }
USE: Write a review with rating
EXAMPLE: tool_rating_review({ toolName: "enterspect_search", review: "Fast and accurate", rating: 5, userId: "user123" })

TOOL: tool_rating_get_top_rated
INPUT: { limit?: number (default: 10), minRatings?: number }
OUTPUT: { topRated: [{ toolName, avgRating, ratingCount }] }
USE: Get highest rated tools
EXAMPLE: tool_rating_get_top_rated({ limit: 5 })

## ANALYTICS (4 tools)

TOOL: tool_analytics_usage
INPUT: { toolName: string (required) }
OUTPUT: { toolName, usageCount, successRate, averageExecutionTime, lastUsed?, firstUsed, trending }
USE: Get usage analytics for a tool
EXAMPLE: tool_analytics_usage({ toolName: "quartermaster_search" })

TOOL: tool_analytics_trends
INPUT: { toolName: string (required), days?: number (default: 30) }
OUTPUT: { toolName, days, data: [{ hourTimestamp, callCount, successCount, errorCount, avgExecutionTime }] }
USE: Get usage trends over time
EXAMPLE: tool_analytics_trends({ toolName: "enterspect_search", days: 7 })

TOOL: tool_analytics_popular
INPUT: { limit?: number (default: 10) }
OUTPUT: { popular: [{ toolName, usageCount }] }
USE: Get most used tools
EXAMPLE: tool_analytics_popular({ limit: 20 })

TOOL: tool_analytics_record_usage
INPUT: { toolName: string (required), success: boolean (required), executionTime?: number, userId?: string }
OUTPUT: { success: boolean, message: string }
USE: Record a tool usage event
EXAMPLE: tool_analytics_record_usage({ toolName: "my_tool", success: true, executionTime: 150 })

## RECOMMENDATIONS (3 tools)

TOOL: tool_recommend_for_me
INPUT: { userId: string (required), limit?: number (default: 5) }
OUTPUT: { recommendations: [{ toolName, reason, confidence }] }
USE: Get personalized tool recommendations
EXAMPLE: tool_recommend_for_me({ userId: "user123" })

TOOL: tool_recommend_similar
INPUT: { toolName: string (required), limit?: number (default: 5) }
OUTPUT: { similar: [{ toolName, similarity, reason }] }
USE: Find tools similar to a given tool
EXAMPLE: tool_recommend_similar({ toolName: "quartermaster_search" })

TOOL: tool_recommend_trending
INPUT: { limit?: number (default: 10), days?: number (default: 7) }
OUTPUT: { trending: [{ toolName, growthRate, currentUsage, previousUsage, percentChange }] }
USE: Get trending tools
EXAMPLE: tool_recommend_trending({ limit: 5 })

## LLM INTELLIGENCE (4 tools)

TOOL: tool_llm_suggest
INPUT: { taskDescription: string (required), requirements?: { accuracy?, urgency?, multimodal?, currentInfo? }, constraints?: object }
OUTPUT: { suggested: { id, name, provider, reasoning, confidence, alternative? }, allOptions: [{ id, name, suitability, pros, cons }] }
USE: Get AI-powered LLM suggestion for a task
EXAMPLE: tool_llm_suggest({ taskDescription: "Analyze an image", requirements: { multimodal: true } })

TOOL: tool_llm_list
INPUT: { provider?: string, availability?: string }
OUTPUT: { llms: [{ id, name, provider, strengths, capabilities, cost, benchmarks, status }] }
USE: List available LLM services
EXAMPLE: tool_llm_list({ provider: "anthropic" })

TOOL: tool_llm_compare
INPUT: { llmIds: string[] (required) }
OUTPUT: { comparison: [{ id, name, strengths, capabilities, cost, benchmarks }] }
USE: Compare multiple LLMs
EXAMPLE: tool_llm_compare({ llmIds: ["claude-3-opus", "gpt-4"] })

TOOL: tool_intelligence_stats
INPUT: {}
OUTPUT: { totalLLMs, providers: [], productionReady }
USE: Get LLM intelligence statistics
EXAMPLE: tool_intelligence_stats({})

## SEARCH INTELLIGENCE (2 tools)

TOOL: tool_search_suggest
INPUT: { requirements?: { needCredibility?, multiSource? } }
OUTPUT: { suggested: { toolName, reasoning, confidence }, alternatives: [{ toolName, suitability }] }
USE: Suggest best search tool for requirements
EXAMPLE: tool_search_suggest({ requirements: { needCredibility: true } })

TOOL: tool_search_compare
INPUT: {}
OUTPUT: { comparison: [{ toolId, speed, credibility, cost }] }
USE: Compare search tools
EXAMPLE: tool_search_compare({})

## PATTERN LEARNING (3 tools)

TOOL: tool_pattern_record_success
INPUT: { taskType: string (required), llmUsed: string (required), executionTime: number (required), context?: object, outcome: string (required) }
OUTPUT: { success: boolean, message: string }
USE: Record successful task completion for pattern learning
EXAMPLE: tool_pattern_record_success({ taskType: "code_review", llmUsed: "claude-3-opus", executionTime: 5000, outcome: "complete" })

TOOL: tool_pattern_get_insights
INPUT: { taskType?: string }
OUTPUT: { totalPatterns, patterns: [{ task, llmUsed, outcomeCount, context }] }
USE: Get learned patterns
EXAMPLE: tool_pattern_get_insights({ taskType: "code_review" })

TOOL: tool_pattern_get_best_llm
INPUT: { taskType: string (required), criteria?: string (default: "successRate") }
OUTPUT: { taskType, bestLLM, confidence, stats: { successCount, avgTime } }
USE: Get best LLM for a task type
EXAMPLE: tool_pattern_get_best_llm({ taskType: "code_review" })

## DEPRECATION (3 tools)

TOOL: tool_deprecate
INPUT: { toolIdentifier: string (required), reason: string (required), removalDate?: string, alternatives?: string[], severity?: string }
OUTPUT: { success: boolean, message: string }
USE: Mark a tool as deprecated
EXAMPLE: tool_deprecate({ toolIdentifier: "old_tool", reason: "Replaced by new_tool", alternatives: ["new_tool"] })

TOOL: tool_deprecation_get_alternatives
INPUT: { toolIdentifier: string (required) }
OUTPUT: { toolIdentifier, alternatives: [], reason }
USE: Get alternatives for a deprecated tool
EXAMPLE: tool_deprecation_get_alternatives({ toolIdentifier: "old_tool" })

TOOL: tool_deprecation_get_migration_guide
INPUT: { fromTool: string (required), toTool: string (required) }
OUTPUT: { fromTool, toTool, steps: [] }
USE: Get migration guide between tools
EXAMPLE: tool_deprecation_get_migration_guide({ fromTool: "old_tool", toTool: "new_tool" })

## COMPATIBILITY (3 tools)

TOOL: tool_compatibility_check
INPUT: { tools: string[] (required) }
OUTPUT: { compatible: boolean, issues: [], recommendations: [] }
USE: Check if tools are compatible together
EXAMPLE: tool_compatibility_check({ tools: ["tool_a", "tool_b"] })

TOOL: tool_compatibility_register_version
INPUT: { toolIdentifier: string (required), version: string (required), requirements?: object, dependencies?: string[] }
OUTPUT: { success: boolean, message: string }
USE: Register a tool version
EXAMPLE: tool_compatibility_register_version({ toolIdentifier: "my_tool", version: "2.0.0" })

TOOL: tool_compatibility_report
INPUT: {}
OUTPUT: { totalVersions, versions: [{ toolId, version, requirements, dependencies, createdAt }] }
USE: Get compatibility report
EXAMPLE: tool_compatibility_report({})

## NOTIFICATIONS (2 tools)

TOOL: tool_notify
INPUT: { title: string (required), message: string (required), category: "deprecation"|"compatibility"|"update"|"security"|"performance"|"general" (required), priority: "low"|"medium"|"high"|"critical" (required) }
OUTPUT: { success: boolean, message: string }
USE: Send a notification
EXAMPLE: tool_notify({ title: "Security Update", message: "New version available", category: "security", priority: "high" })

TOOL: tool_notifications_get
INPUT: { category?: string, priority?: "low"|"medium"|"high"|"critical", unreadOnly?: boolean, limit?: number (default: 20) }
OUTPUT: { notifications: [{ id, title, message, category, priority, createdAt, read }] }
USE: Get notifications
EXAMPLE: tool_notifications_get({ unreadOnly: true })

## LEARNING SYSTEM (3 tools)

TOOL: tool_learning_record_interaction
INPUT: { type: string (required), toolId?: uuid, outcome: string (required), context?: object }
OUTPUT: { success: boolean }
USE: Record a learning interaction
EXAMPLE: tool_learning_record_interaction({ type: "tool_use", outcome: "success" })

TOOL: tool_learning_get_recommendations
INPUT: {}
OUTPUT: { recommendations: [{ toolName, confidence, reason }] }
USE: Get learned recommendations
EXAMPLE: tool_learning_get_recommendations({})

TOOL: tool_learning_get_insights
INPUT: {}
OUTPUT: { totalInteractions, recentPatterns: [] }
USE: Get learning insights
EXAMPLE: tool_learning_get_insights({})

## STATUS & HEALTH (3 tools)

TOOL: tool_status_dashboard
INPUT: {}
OUTPUT: { tools: { total, enabled, healthy }, servers: { total, healthy }, timestamp }
USE: Get status dashboard
EXAMPLE: tool_status_dashboard({})

TOOL: tool_status_summary
INPUT: {}
OUTPUT: { healthScore, totalTools, enabledTools, serversTracked, serversHealthy, recentIssues, systemStatus }
USE: Get health summary
EXAMPLE: tool_status_summary({})

TOOL: tool_status_health_check
INPUT: {}
OUTPUT: { overall, components: [{ component, status, responseTime }], servers: [{ serverId, status }] }
USE: Run health check
EXAMPLE: tool_status_health_check({})

## ECOSYSTEM (5 tools)

TOOL: tool_ecosystem_servers_list
INPUT: { type?: string, status?: "healthy"|"degraded"|"down" }
OUTPUT: { servers: [{ serverId, name, type, version, port, status, toolCount, capabilities }] }
USE: List ecosystem servers
EXAMPLE: tool_ecosystem_servers_list({ status: "healthy" })

TOOL: tool_ecosystem_server_get
INPUT: { serverId: string (required) }
OUTPUT: { serverId, name, type, status, toolCount, lastHeartbeat }
USE: Get server details
EXAMPLE: tool_ecosystem_server_get({ serverId: "quartermaster" })

TOOL: tool_ecosystem_map
INPUT: {}
OUTPUT: { nodes: [{ id, name, type }], connections: [] }
USE: Get ecosystem map
EXAMPLE: tool_ecosystem_map({})

TOOL: tool_ecosystem_health
INPUT: {}
OUTPUT: { overallHealth, healthyServers, totalServers }
USE: Get ecosystem health
EXAMPLE: tool_ecosystem_health({})

TOOL: tool_ecosystem_statistics
INPUT: {}
OUTPUT: { totalTools, totalServers, averageToolsPerServer, categories }
USE: Get ecosystem statistics
EXAMPLE: tool_ecosystem_statistics({})

## OPTIMIZATION (4 tools)

TOOL: tool_ecosystem_coordinate
INPUT: { operation: string (required), servers: string[] (required), params?: object }
OUTPUT: { success: boolean, message: string }
USE: Coordinate operation across servers
EXAMPLE: tool_ecosystem_coordinate({ operation: "reindex", servers: ["quartermaster", "enterspect"] })

TOOL: tool_optimizer_track_operation
INPUT: { operationId: uuid (required), metrics: object (required) }
OUTPUT: { success: boolean, operationId }
USE: Track optimization metrics
EXAMPLE: tool_optimizer_track_operation({ operationId: "abc-123", metrics: { duration: 500 } })

TOOL: tool_optimizer_suggest
INPUT: { targetMetric?: string }
OUTPUT: { suggestions: [] }
USE: Get optimization suggestions
EXAMPLE: tool_optimizer_suggest({ targetMetric: "latency" })

TOOL: tool_optimizer_statistics
INPUT: {}
OUTPUT: { totalOptimizations, averageImprovement }
USE: Get optimization statistics
EXAMPLE: tool_optimizer_statistics({})

## INTEGRATION ADAPTERS (6 tools)

TOOL: tool_guardian_validate
INPUT: { operation: string (required), params?: object }
OUTPUT: { approved: boolean, reason, riskScore }
USE: Validate operation through context-guardian
EXAMPLE: tool_guardian_validate({ operation: "file_delete", params: { path: "/important" } })

TOOL: tool_quartermaster_recommend
INPUT: { path: string (required), action?: string }
OUTPUT: { recommendedTools: [], reasoning }
USE: Get quartermaster recommendations
EXAMPLE: tool_quartermaster_recommend({ path: "/documents/report.md" })

TOOL: tool_search_routing_suggest
INPUT: { query: string (required), context?: object }
OUTPUT: { suggestedTool, confidence }
USE: Suggest search routing
EXAMPLE: tool_search_routing_suggest({ query: "find config files" })

TOOL: tool_glec_classify
INPUT: { content: string (required), contentType?: string }
OUTPUT: { toolName, classification: "Genesis"|"Leviticus"|"Exodus"|"Context", confidence }
USE: GLEC classify content
EXAMPLE: tool_glec_classify({ content: "Project requirements document" })

TOOL: tool_glec_bulk_classify
INPUT: { tools: string[] (required) }
OUTPUT: { results: [{ toolName, classification, confidence }] }
USE: Bulk GLEC classification
EXAMPLE: tool_glec_bulk_classify({ tools: ["tool_a", "tool_b"] })

TOOL: discover_tools
INPUT: { force?: boolean }
OUTPUT: { totalDiscovered, byServer: {}, newTools: [], updatedTools: [], duration }
USE: Discover tools from all connected servers
EXAMPLE: discover_tools({ force: true })

## CLAUDE INTEGRATION (3 tools)

TOOL: tool_claude_desktop_capabilities
INPUT: { capability?: string }
OUTPUT: { capabilities: object } or { name, description, version }
USE: Get Claude Desktop capabilities
EXAMPLE: tool_claude_desktop_capabilities({})

TOOL: tool_claude_desktop_updates
INPUT: {}
OUTPUT: { current_version, updates: [], latest_additions: [] }
USE: Get Claude Desktop updates
EXAMPLE: tool_claude_desktop_updates({})

TOOL: tool_remind_claude_capabilities
INPUT: {}
OUTPUT: { reminder, relevant_tools: [{ tool_name, description }], suggestion }
USE: Remind Claude of its capabilities
EXAMPLE: tool_remind_claude_capabilities({})

---

LAYERS

1. MCP stdio (65 tools) - Primary interface
2. InterLock UDP mesh (port 3004) - Peer communication
3. HTTP REST API (port 8004) - External integrations
4. WebSocket real-time (port 9004) - Live updates

---

SERVICES

- Registry: Core tool management (add, update, delete, list)
- Capability: Natural language "can do" matching
- Curation: Quality control and flagging
- Rating: User ratings and reviews
- Analytics: Usage tracking and trends
- Recommendations: Personalized suggestions
- LLM Intelligence: AI model selection
- Pattern Learning: Success pattern tracking
- Deprecation: Tool lifecycle management
- Compatibility: Version and dependency tracking
- Notifications: Alert system
- Learning: Continuous improvement
- Status: Health monitoring
- Ecosystem: Server coordination
- Optimization: Performance tuning
- Integration: Cross-server adapters
- Claude: Desktop integration

---

KEY FILES

SOURCE: /repo/Toolee/Tool_Registry/
INDEX: src/index.js
MCP: src/mcp/server.js
TOOLS: src/tools/all-tools.js (65 tools)
SCHEMAS: src/tools/schemas.js
SERVICES: src/services/*.js
INTERLOCK: src/mesh/socket.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
DATABASE: src/database/client.js, schema.js
CONFIG: config/server.json, config/interlock.json

DEPENDENCIES: @modelcontextprotocol/sdk, better-sqlite3, express, ws, zod, langfuse, redis, uuid, node-cron
