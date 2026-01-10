SERVER: niws-delivery
VERSION: 1.0
UPDATED: 2026-01-10
STATUS: Production
PORTS: UDP 3036 | HTTP 8036 | WS 9036
MCP: stdio transport (stdin/stdout JSON-RPC)
INTERLOCK: Enabled (peer mesh communication, receives all signals)
DEPS: Notion API, FFmpeg (video), niws-production (scripts/briefs), AirDrop (macOS)
PURPOSE: Final delivery pipeline - export, video production, Notion workflow, automated scheduling
CONFIG: ENV HTTP_PORT (default 8036), NOTION_API_KEY, NOTION_DATABASE_ID, EXPORT_DIR

---

ARCHITECTURE

FUNCTION: Fourth stage of NIWS pipeline - final delivery and automation
FLOW: Approved Scripts -> Export/Video -> Notion Workflow -> Distribution
STORAGE: SQLite (delivery.db), File exports (data/exports/)
COMPONENTS:
  - Notion integration: Story approval workflow
  - Teleprompter export: RTF/HTML/TXT for iPad
  - Video pipeline: FFmpeg-based processing
  - Orchestrator: Automated overnight/morning workflows
KEY_FEATURE: Fully automated end-to-end delivery with approval gates

---

TOOL CATEGORIES

NOTION (12 tools):
- notion_push_story: Push story with brief and script to Notion database
- notion_poll_approvals: Poll for approved stories ready for export
- notion_mark_story_sent: Mark story as sent/exported
- notion_get_approved_stories: Get all stories with Approved status
- notion_create_page: Create new Notion page
- notion_update_page: Update existing page properties
- notion_archive_page: Archive a Notion page
- notion_get_database: Get database structure and properties
- notion_sync_status: Check Notion sync connection status
- notion_get_comments: Get comments on a page
- notion_add_comment: Add comment to a page
- notion_get_workflow_state: Get workflow state (active, pending, completed)

TELEPROMPTER (6 tools):
- export_teleprompter: Export script to RTF/HTML/TXT with theme options
- airdrop_to_ipad: Send file to iPad via AirDrop (macOS only)
- batch_export_and_transfer: Export multiple scripts and optionally AirDrop
- cleanup_exports: Clean up old export files
- export_rtf: Export script to RTF format
- export_html: Export script to HTML with auto-scroll

VIDEO (14 tools):
- build_video_background: Generate video background for script
- composite_final_video: Composite from background and foreground
- export_all_platforms: Export to multiple platforms (YouTube, TikTok, Instagram, Twitter)
- run_video_pipeline: Run full video pipeline
- apply_chroma_key: Apply green/blue screen effect
- add_motion_graphics: Add lower thirds, title cards, news tickers
- create_pip_composite: Create picture-in-picture composite
- capture_scroll: Create scrolling video from tall image
- encode_video: Encode with preset settings
- preview_video: Generate preview clip
- get_video_status: Get processing job status
- cancel_video_job: Cancel video job
- get_video_assets: List available video assets
- cleanup_video_temp: Clean up temporary video files

ORCHESTRATOR (16 tools):
- start_overnight_run: Start overnight automation workflow
- start_morning_poll: Start morning poll workflow
- get_workflow_status: Get current workflow status
- pause_workflow: Pause running workflow
- resume_workflow: Resume paused workflow
- cancel_workflow: Cancel current workflow
- schedule_workflow: Schedule workflow with cron expression
- get_schedule: Get all scheduled workflows
- update_schedule: Update workflow schedule
- get_workflow_logs: Get execution logs
- get_workflow_metrics: Get performance metrics
- notify_completion: Send desktop notification
- get_pending_actions: Get actions requiring attention
- approve_action: Approve pending action
- reject_action: Reject pending action
- get_workflow_history: Get historical workflow runs

TOTAL: 48 tools (12 notion + 6 teleprompter + 14 video + 16 orchestrator)

---

TELEPROMPTER FORMATS

FORMAT: rtf
DESC: Rich Text Format for professional teleprompter apps
THEMES: light | dark | sepia
FEATURES: Font sizing, color themes, page breaks

FORMAT: html
DESC: HTML with optional auto-scroll for browser-based prompting
THEMES: light | dark
FEATURES: Auto-scroll, dark mode, responsive sizing

FORMAT: txt
DESC: Plain text with uppercase conversion
FEATURES: Uppercase, line breaks, clean margins

---

VIDEO PIPELINE

PLATFORMS: youtube | tiktok | instagram | twitter
QUALITY_PRESETS: fast | balanced | quality
CHROMA_COLORS: green | blue | magenta

MOTION_GRAPHICS:
- lower_third: Name/title overlays
- title_card: Full-screen title cards
- news_ticker: Scrolling news ticker

PIPELINE_STEPS:
1. Build video background from script
2. Apply chroma key (optional)
3. Add motion graphics
4. Composite PiP if needed
5. Encode for target platforms
6. Export to platform specs

---

WORKFLOW TYPES

TYPE: overnight
DESC: Full automation - poll feeds, cluster stories, generate briefs/scripts
SCHEDULE: Cron-based (e.g., "0 2 * * *" for 2 AM daily)
STEPS: Feed poll -> Clustering -> Brief generation -> Script generation -> Push to Notion

TYPE: morning
DESC: Review and approval workflow
SCHEDULE: Cron-based (e.g., "0 7 * * *" for 7 AM daily)
STEPS: Poll Notion approvals -> Export teleprompter -> Notify completion

TYPE: hourly_poll
DESC: Lightweight feed polling
SCHEDULE: Cron-based (e.g., "0 * * * *" for hourly)
STEPS: Feed poll only, no processing

---

NOTION WORKFLOW STATES

STATE: draft
DESC: Story pushed, awaiting review
TRANSITIONS_TO: review

STATE: review
DESC: Under editorial review
TRANSITIONS_TO: approved | rejected

STATE: approved
DESC: Ready for export and delivery
TRANSITIONS_TO: sent

STATE: sent
DESC: Exported and delivered

STATE: rejected
DESC: Not approved for publication

---

DATABASE SCHEMA

TABLE: exports
COLUMNS: id, script_id, format, file_path, theme, created_at

TABLE: video_jobs
COLUMNS: id, script_id, platforms, status, progress, output_path, error, created_at, completed_at

TABLE: workflow_runs
COLUMNS: id, workflow_type, status, current_step, started_at, completed_at

TABLE: workflow_logs
COLUMNS: id, run_id, level, message, timestamp

TABLE: schedules
COLUMNS: id, workflow_type, cron_expr, enabled, next_run, last_run

TABLE: pending_actions
COLUMNS: id, action_type, data, status, created_at

---

HTTP ENDPOINTS

GET /api/health - Health check
GET /api/status - Server status with dependency health

GET /api/exports - List exports
POST /api/exports/teleprompter - Export to teleprompter format
POST /api/exports/airdrop - Send file via AirDrop

GET /api/video/jobs - List video jobs
POST /api/video/pipeline - Start video pipeline
GET /api/video/jobs/:id - Get job status
DELETE /api/video/jobs/:id - Cancel job
GET /api/video/assets - List video assets

GET /api/workflow/status - Current workflow status
POST /api/workflow/start/:type - Start workflow (overnight, morning)
POST /api/workflow/pause - Pause workflow
POST /api/workflow/resume - Resume workflow
POST /api/workflow/cancel - Cancel workflow
GET /api/workflow/logs - Get logs
GET /api/workflow/history - Get history

GET /api/notion/status - Notion sync status
POST /api/notion/push - Push story to Notion
GET /api/notion/approvals - Get approved stories
GET /api/notion/workflow - Get workflow state

GET /api/schedules - Get schedules
PUT /api/schedules/:id - Update schedule

---

WEBSOCKET EVENTS

EVENT: export:started - Export begun
EVENT: export:completed - Export finished
EVENT: video:queued - Video job queued
EVENT: video:processing - Video processing
EVENT: video:completed - Video finished
EVENT: workflow:started - Workflow begun
EVENT: workflow:step - Workflow step completed
EVENT: workflow:completed - Workflow finished
EVENT: notion:pushed - Story pushed to Notion
EVENT: notion:approved - Story approved in Notion

---

INTERLOCK SIGNALS

EMIT:
- server:ready - Server operational
- server:shutdown - Server shutting down
- export:started - Export begun
- export:completed - Export finished
- video:queued - Video job queued
- video:processing - Video being processed
- video:completed - Video finished
- workflow:started - Workflow begun
- workflow:step - Workflow step
- workflow:completed - Workflow finished
- notion:pushed - Story pushed to Notion
- notion:approved - Story approved

RECEIVE:
- * (receives all signals from mesh)

PEERS: niws-intake (3033), niws-analysis (3034), niws-production (3035), context-guardian (3001), consciousness-mcp (3028)

---

SERVICE CLIENTS

CLIENT: intakeClient
URL: http://localhost:8033
METHODS: getStory, getArticlesForStory, listStories

CLIENT: productionClient
URL: http://localhost:8035
METHODS: getScript, getBrief, listScripts, listBriefs

CLIENT: notionClient
URL: https://api.notion.com
METHODS: pushStory, pollApprovals, markStorySent, createPage, updatePage, archivePage, getComments, addComment

---

KEY FILES

SOURCE: /repo/niws-delivery/
INDEX: src/index.ts
HTTP: src/http/server.ts
WS: src/websocket/server.ts
INTERLOCK: src/interlock/
TOOLS:
  - src/tools/notion-tools.ts (12 tools)
  - src/tools/teleprompter-tools.ts (6 tools)
  - src/tools/video-tools.ts (14 tools)
  - src/tools/orchestrator-tools.ts (16 tools)
SERVICES:
  - src/services/notionClient.ts
  - src/services/clients.ts
  - src/services/notifications.ts
  - src/services/airdrop.ts
  - src/services/teleprompterFormatter.ts
  - src/services/videoOrchestrator.ts
VIDEO:
  - src/video/chromaKey.ts
  - src/video/motionGraphics.ts
  - src/video/pipCompositor.ts
  - src/video/multiPlatformExport.ts
  - src/video/scrollCapture.ts
ORCHESTRATOR:
  - src/orchestrator/stateManager.ts
  - src/orchestrator/overnight.ts
  - src/orchestrator/morningPoll.ts
  - src/orchestrator/scheduler.ts
EXPORTERS:
  - src/exporters/rtf.ts
  - src/exporters/html.ts
  - src/exporters/plainText.ts
CONFIG:
  - config/interlock.json
  - src/config/videoConfig.ts
DATA: data/delivery.db, data/exports/
