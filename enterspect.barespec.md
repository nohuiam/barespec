SERVER: enterspect
VERSION: 2.0
UPDATED: 2025-12-26
STATUS: Production
PORT: 3009 (UDP/InterLock), 8009 (HTTP), 9009 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: Internal semantic and keyword search across BoxOfPrompts filesystem
CONFIG: /repo/EnterSpect/config/interlock.json

---

ARCHITECTURE

SEARCH: Semantic (NL) + Keyword (traditional) + Spatial (3D)
INDEX: In-memory with persistence, GLEC-aware scoping
DATABASE: SQLite (better-sqlite3)
LAYERS: MCP stdio (8 tools), InterLock UDP mesh, HTTP REST, WebSocket events
FUTURE: VRirlPool 3D visualization interface

---

TOOLS (8)

TOOL: semantic_search
INPUT: { query: string (required), scope?: "all"|"genesis"|"leviticus"|"exodus"|"context" (default: "all"), max_results?: number (default: 50) }
OUTPUT: { results: [{ path, filename, score, snippet, metadata }], total, search_time_ms }
USE: Search files using natural language queries with semantic understanding
EXAMPLE: semantic_search({ query: "authentication flow architecture", scope: "genesis", max_results: 20 })

TOOL: keyword_search
INPUT: { keywords: string[] (required), file_types?: string[] (default: []), date_range?: { start: number, end: number }, max_results?: number (default: 100) }
OUTPUT: { results: [{ path, filename, matches, score }], total, search_time_ms }
USE: Search files using traditional keyword matching
EXAMPLE: keyword_search({ keywords: ["GLEC", "classification"], file_types: [".md"] })

TOOL: filter_by_category
INPUT: { glec_type: "genesis"|"leviticus"|"exodus"|"context" (required), subject?: string, importance?: number (1-5) }
OUTPUT: { success: boolean, glec_type, total, results: [] }
USE: Filter files by GLEC category and metadata
EXAMPLE: filter_by_category({ glec_type: "genesis", importance: 4 })

TOOL: recent_activity
INPUT: { activity_type?: "modified"|"accessed"|"created" (default: "modified"), limit?: number (default: 50), time_range_hours?: number (default: 24) }
OUTPUT: { results: [{ path, filename, timestamp, activity_type }], total }
USE: Get recently modified, accessed, or created files
EXAMPLE: recent_activity({ activity_type: "modified", time_range_hours: 6 })

TOOL: spatial_navigate
INPUT: { x: number (required), y: number (required), z: number (required), radius?: number (default: 10) }
OUTPUT: { success: boolean, center: { x, y, z }, radius, total, results: [{ file_path, position, color, size_factor }] }
USE: Navigate files spatially in 3D space (future VRirlPool interface)
EXAMPLE: spatial_navigate({ x: 0, y: 0, z: 0, radius: 25 })

TOOL: find_related
INPUT: { file_path: string (required), similarity_threshold?: number (default: 0.5, 0-1), max_results?: number (default: 20) }
OUTPUT: { success: boolean, reference_file, total, results: [{ path, score, snippet }] }
USE: Find documents semantically related to a given file
EXAMPLE: find_related({ file_path: "/path/to/reference.md", similarity_threshold: 0.7 })

TOOL: search_index_status
INPUT: {}
OUTPUT: { success: boolean, index_health, total_documents, last_update, index_size }
USE: Get search index health and statistics
EXAMPLE: search_index_status({})

TOOL: update_search_index
INPUT: { full_rebuild?: boolean (default: false), path?: string }
OUTPUT: { success: boolean, files_indexed, index_size, timestamp }
USE: Manually refresh the search index
EXAMPLE: update_search_index({ full_rebuild: false, path: "/Documents" })

---

SEARCH MODES

- Semantic: Natural language queries using embeddings
- Keyword: Traditional keyword matching (all keywords must match)
- Spatial: 3D navigation (VRirlPool future)

---

KEY FILES

SOURCE: /repo/EnterSpect/
INDEX: index.js
TOOLS: src/tools/index.js
SEARCH: src/search/semantic-search.js, src/search/keyword-search.js
CORE: src/core/operations.js
DATABASE: src/database/db.js
INTERLOCK: src/interlock/socket.js, src/interlock/protocol.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
CONFIG: config/server.json, config/interlock.json

DEPENDENCIES: @modelcontextprotocol/sdk, better-sqlite3, express, ws
