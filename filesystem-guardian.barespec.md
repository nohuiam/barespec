SERVER: filesystem-guardian
VERSION: 1.0.0
UPDATED: 2026-01-02
STATUS: Tested (6 tests passed)
PORT: 3026 (UDP/InterLock), 8026 (HTTP), 9026 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: better-sqlite3, ws, uuid, zod, @modelcontextprotocol/sdk
PURPOSE: macOS-native filesystem operations - extended attributes (xattr) and Spotlight search
CONFIG: /repo/filesystem-guardian/config/interlock.json
TESTS: 6 (xattr, spotlight, watch, HTTP, InterLock)

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3) with 2 tables
PLATFORM: macOS-native (xattr CLI, mdfind, mdimport, fs.watch)
WORKFLOW: Command → Native CLI → Parse Result → Return/Broadcast
NOTE: Some features limited on non-macOS systems

---

TOOLS (6)

TOOL: get_xattr
INPUT: { path: string (required), attribute?: string }
OUTPUT: { path: string, attributes: object }
USE: Read extended attributes from a file
EXAMPLE: get_xattr({ path: "/path/to/file.md", attribute: "com.apple.metadata:tags" })
NOTES: Returns all xattrs if attribute not specified

TOOL: set_xattr
INPUT: { path: string (required), attribute: string (required), value: string|null (required), create_only?: boolean }
OUTPUT: { success: boolean, path: string, attribute: string }
USE: Write or delete extended attributes (null value = delete)
EXAMPLE: set_xattr({ path: "/path/to/file.md", attribute: "user.custom", value: "my-value" })
NOTES: create_only prevents overwriting existing attributes

TOOL: list_xattr
INPUT: { path: string (required) }
OUTPUT: { path: string, attributes: string[] }
USE: List all extended attribute names on a file
EXAMPLE: list_xattr({ path: "/path/to/file.md" })

TOOL: spotlight_search
INPUT: { query: string (required), scope?: string, limit?: number }
OUTPUT: { results: [], count: number, query: string }
USE: Query Spotlight index with MDQuery syntax
EXAMPLE: spotlight_search({ query: "kMDItemDisplayName == '*.md'", scope: "/Users/macbook/Documents", limit: 100 })
NOTES: MDQuery syntax - see Apple documentation

TOOL: spotlight_reindex
INPUT: { path: string (required) }
OUTPUT: { success: boolean, path: string, message: string }
USE: Force Spotlight to reindex a path
EXAMPLE: spotlight_reindex({ path: "/Users/macbook/Documents/project" })
NOTES: Calls mdimport -d1 on the path

TOOL: watch_volume
INPUT: { path: string (required), recursive?: boolean }
OUTPUT: { watch_id: string, path: string, active: boolean }
USE: Monitor filesystem events at volume/directory level
EXAMPLE: watch_volume({ path: "/Users/macbook/Documents", recursive: true })
NOTES: Events broadcast via WebSocket, subscribe with watch_id

---

LAYERS

1. MCP stdio (6 tools) - Primary interface
2. InterLock UDP mesh (port 3026) - Peer communication
3. HTTP REST API (port 8026) - External integrations
4. WebSocket real-time (port 9026) - Live updates

---

HTTP REST API

GET  /health                → { status, active_watches, operations_count }
GET  /api/watches           → List active watches
GET  /api/operations        → Get recent xattr operations
POST /api/xattr/get         → Get xattr values
POST /api/xattr/set         → Set xattr values
POST /api/xattr/list        → List xattr names
POST /api/spotlight/search  → Query Spotlight
POST /api/spotlight/reindex → Trigger reindex
POST /api/watch/start       → Start a watch
POST /api/watch/stop        → Stop a watch

---

WEBSOCKET EVENTS

S→C connected               → Initial connection confirmation
S→C fs_event                → Filesystem event broadcast
C→S subscribe               → Subscribe to watch_id
C→S unsubscribe             → Unsubscribe from watch_id
C↔S ping/pong               → Keepalive

---

INTERLOCK SIGNALS

Handlers (7):
- ping → pong with timestamp
- status_request → Server status and watch count
- xattr_query → Xattr values for path
- xattr_list → Xattr names for path
- spotlight_query → Spotlight search results
- watch_list → Active watches list
- file_metadata_request → Path metadata including xattrs

PEERS: quartermaster (3002), catasorter (3005), smart-file-organizer (3007), enterspect (3009), context-guardian (3001), trinity-coordinator (3012)

---

DATABASE SCHEMA

TABLE: xattr_operations
COLUMNS: id, path, operation, attribute, value, timestamp, success
INDEXES: idx_xattr_path, idx_xattr_timestamp

TABLE: fs_watches
COLUMNS: id, watch_id, path, recursive, created_at, active
INDEXES: idx_watches_id, idx_watches_path

---

KEY FILES

SOURCE: /repo/filesystem-guardian/
INDEX: src/index.ts
SERVICES: src/services/ (xattr-service.ts, spotlight-service.ts, fsevents-service.ts)
TOOLS: src/tools/
INTERLOCK: src/interlock/socket.ts (port 3026)
HTTP: src/http/server.ts (port 8026)
WEBSOCKET: src/websocket/server.ts (port 9026)
DATABASE: data/filesystem-guardian.db
CONFIG: config/interlock.json

DEPENDENCIES: better-sqlite3, ws, uuid, zod, @modelcontextprotocol/sdk

---

SERVICES

- XattrService: Extended attribute operations via `xattr` CLI
- SpotlightService: Spotlight search via `mdfind`/`mdimport`
- FSEventsService: Filesystem watching via `fs.watch`

---

MACOS NATIVE COMMANDS

| Command | Purpose |
|---------|---------|
| xattr | Extended attribute read/write/delete |
| mdfind | Spotlight search queries |
| mdimport | Force Spotlight reindexing |
| fs.watch | Filesystem event monitoring |

---

MDQUERY SYNTAX EXAMPLES

```
kMDItemDisplayName == "*.md"           # Files by name
kMDItemTextContent == "*keyword*"      # Content search
kMDItemContentType == "public.folder"  # Folders only
kMDItemFSCreationDate > $time.today    # Created today
```

---

NOTES

1. macOS-specific - uses native CLI tools
2. xattr operations logged for audit trail
3. Spotlight queries use MDQuery syntax
4. Filesystem watches broadcast via WebSocket
5. Limited functionality on non-macOS systems
6. Integrates with smart-file-organizer for file ops
