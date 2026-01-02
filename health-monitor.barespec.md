SERVER: health-monitor
VERSION: 1.0.0
UPDATED: 2026-01-02
STATUS: Tested (24 tests passed)
PORT: 3024 (UDP/InterLock), 8024 (HTTP), 9024 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk
PURPOSE: Ecosystem health monitoring with alerts, metrics, and predictive failure analysis
CONFIG: /repo/health-monitor/config/interlock.json
TESTS: 24 (startup, HTTP, health monitoring, alerts, websocket, error handling)

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3) with 5 tables
WORKFLOW: Scheduler → Health Check → Analyze → Alert/Predict → Broadcast
INTERVAL: 30 seconds (configurable)
RETENTION: 168 hours (configurable)
MONITORS: 22 servers in ecosystem

---

TOOLS (7)

TOOL: check_health
INPUT: { servers?: string[] }
OUTPUT: { servers: [], summary: object }
USE: Check health of all or specified monitored servers
EXAMPLE: check_health({ servers: ["context-guardian", "quartermaster"] })

TOOL: get_server_status
INPUT: { server_name: string (required) }
OUTPUT: { server: string, status: string, last_check: string, metrics: object, alerts: [] }
USE: Get detailed status for a specific server
EXAMPLE: get_server_status({ server_name: "bonzai-bloat-buster" })

TOOL: acknowledge_alert
INPUT: { alert_id: string (required), acknowledged_by?: string }
OUTPUT: { acknowledged: boolean, alert_id: string, timestamp: string }
USE: Acknowledge an active alert
EXAMPLE: acknowledge_alert({ alert_id: "alert-123" })

TOOL: get_metrics
INPUT: { server_name?: string, metric_type?: string }
OUTPUT: { metrics: [], aggregated: object }
USE: Get real-time metrics for servers
EXAMPLE: get_metrics({ server_name: "trinity-coordinator" })

TOOL: get_aggregated_health
INPUT: { }
OUTPUT: { total_servers: number, healthy_count: number, degraded_count: number, critical_count: number, offline_count: number, ecosystem_status: string }
USE: Get ecosystem health summary
EXAMPLE: get_aggregated_health({})

TOOL: subscribe_to_alerts
INPUT: { subscription_id?: string, filter?: object }
OUTPUT: { subscription_id: string, created: boolean }
USE: Subscribe to real-time alert streaming via WebSocket
EXAMPLE: subscribe_to_alerts({ filter: { severity: "critical" } })

TOOL: get_server_history
INPUT: { server_name: string (required), resolution?: string, hours?: number }
OUTPUT: { server: string, history: [], resolution: string }
USE: Get historical health data for charts/trends
EXAMPLE: get_server_history({ server_name: "snapshot", resolution: "hourly", hours: 24 })

---

LAYERS

1. MCP stdio (7 tools) - Primary interface
2. InterLock UDP mesh (port 3024) - Peer communication
3. HTTP REST API (port 8024) - External integrations
4. WebSocket real-time (port 9024) - Live updates

---

HTTP REST API

GET  /health                    → { status, scheduler_active, stats }
GET  /api/health                → All 22 servers status
GET  /api/servers/:name         → Individual server status
GET  /api/servers/:name/history → Server health history
GET  /api/alerts                → All alerts
GET  /api/alerts/active         → Active alerts count
POST /api/alerts/:id/ack        → Acknowledge alert
GET  /api/metrics               → Metrics for all servers
GET  /api/predictions           → Predictive failure alerts
GET  /api/summary               → Ecosystem summary

---

WEBSOCKET EVENTS

S→C health_alert              → Alert triggered
S→C server_status_changed     → Server status change
S→C metric_threshold_exceeded → Threshold violation
S→C prediction_alert          → Predictive failure warning
S→C health_check_complete     → Health check cycle done
S→C error                     → Error occurred
C↔S ping/pong                 → Keepalive

---

INTERLOCK SIGNALS

Emits: HEALTH_ALERT, STATUS_CHANGED, METRIC_THRESHOLD, PREDICTION_ALERT, HEALTH_SUMMARY
Accepts: HEALTH_CHECK_REQUEST, STATUS_QUERY, ACKNOWLEDGE_ALERT, PING, HEARTBEAT

---

DATABASE SCHEMA

TABLE: health_checks
COLUMNS: id, server_name, status, response_time_ms, cpu_percent, memory_mb, checked_at
INDEXES: idx_health_server, idx_health_date

TABLE: health_alerts
COLUMNS: id, server_name, alert_type, severity, category, message, recommended_action, created_at, acknowledged_at, acknowledged_by
INDEXES: idx_alerts_server, idx_alerts_severity, idx_alerts_active

TABLE: predictive_alerts
COLUMNS: id, server_name, prediction_type, probability, predicted_failure_time, created_at
INDEXES: idx_predictions_server

TABLE: metric_history
COLUMNS: id, server_name, metric_type, value, recorded_at
INDEXES: idx_metrics_server, idx_metrics_type

TABLE: alert_subscriptions
COLUMNS: id, subscription_id, filter_json, created_at
INDEXES: idx_subscriptions_id

---

KEY FILES

SOURCE: /repo/health-monitor/
INDEX: src/index.ts
HEALTH: src/health/ (checker.ts, analyzer.ts, predictor.ts, scheduler.ts)
TOOLS: src/tools/
INTERLOCK: src/interlock/socket.ts
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
DATABASE: data/health-monitor.db
CONFIG: config/interlock.json, config/servers.json, config/thresholds.json

DEPENDENCIES: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk

---

SERVICES

- HealthChecker: Pings servers, collects response times
- AlertAnalyzer: Generates alerts for threshold violations
- TrendPredictor: Analyzes patterns, predicts failures
- HealthScheduler: Runs checks at configurable interval

---

THRESHOLDS (Default)

| Metric | Warning | Critical |
|--------|---------|----------|
| Response Time | >500ms | >2000ms |
| CPU | >70% | >90% |
| Memory | >80% | >95% |

---

NOTES

1. Monitors 22 servers in the ecosystem
2. Health check interval: 30 seconds (configurable)
3. History retention: 168 hours (configurable)
4. Prediction threshold: 70% probability
5. Auto-cleanup of old data
6. Self-monitoring (reports own health)
