SERVER: niws-server
VERSION: 2.1
UPDATED: 2025-12-27
STATUS: Production
PORT: 8015 (HTTP only)
MCP: stdio transport (stdin/stdout JSON-RPC)
UDP: None (no InterLock mesh)
WS: None
DEPS: Notion API
PURPOSE: News Intelligence Workflow System - complete news production pipeline from RSS ingestion to video delivery
CONFIG: ENV ADMIN_HTTP_PORT (default 8015)

---

ARCHITECTURE

FUNCTION: End-to-end news production pipeline
FLOW: RSS Ingestion → Clustering → Analysis → Scripting → Approval → Production → Delivery
STORAGE: SQLite (outlets, feeds, articles, clusters, scripts)
APPROVAL: Notion integration for story approval workflow
DELIVERY: Teleprompter export, AirDrop, video compositing

---

TOOLS (102)

## CORE RESEARCH (4)

TOOL: research_query
INPUT: { query: string (required), backend?: "auto"|"direct_perplexity"|"comet_flow" (default: "auto"), model?: "sonar-small"|"sonar"|"sonar-pro" (default: "sonar-small"), maxTokens?: number (default: 1000), importance?: "low"|"medium"|"high" (default: "medium"), searchDomainFilter?: string[], searchRecencyFilter?: "day"|"week"|"month"|"year", returnCitations?: boolean (default: true) }
OUTPUT: { answer: string, citations: array, backend_used: string, tokens_used: number, quality_score: number, cached: boolean }
USE: Execute research via best available backend
EXAMPLE: research_query({ query: "latest AI agent developments 2025", importance: "high" })
NOTES: "auto" selects backend based on query complexity and budget.

TOOL: get_research_stats
INPUT: { time_range?: "hour"|"day"|"week"|"month" }
OUTPUT: { queries_total: number, backend_usage: object, avg_quality_score: number, cache_hit_rate: number, budget_used: number }
USE: Get research query statistics
EXAMPLE: get_research_stats({ time_range: "week" })

TOOL: get_budget_status
INPUT: {}
OUTPUT: { daily_budget: number, used_today: number, remaining: number, reset_time: timestamp }
USE: Get current budget status
EXAMPLE: get_budget_status({})

TOOL: get_cache_stats
INPUT: {}
OUTPUT: { total_cached: number, hit_rate: number, size_bytes: number, oldest_entry: timestamp }
USE: Get cache performance statistics
EXAMPLE: get_cache_stats({})

## OUTLET DATABASE (10)

TOOL: outlet_add - Add news outlet
TOOL: outlet_search - Search outlets
TOOL: outlet_get - Get outlet details
TOOL: outlet_update - Update outlet
TOOL: outlet_delete - Delete outlet
TOOL: outlet_list_by_type - List by type
TOOL: outlet_list_by_bias - List by bias
TOOL: outlet_get_stats - Get statistics
TOOL: outlet_import - Bulk import
TOOL: outlet_export - Export database

## RSS BACKEND (13)

TOOL: rss_add_feed - Add RSS feed
TOOL: rss_remove_feed - Remove feed
TOOL: rss_get_feed - Get feed details
TOOL: rss_list_feeds - List feeds
TOOL: rss_poll_feeds - Poll all feeds
TOOL: rss_poll_single - Poll single feed
TOOL: rss_get_articles - Get articles
TOOL: rss_set_poll_interval - Set interval
TOOL: rss_get_poll_status - Get status
TOOL: rss_mark_article_read - Mark read
TOOL: rss_get_unread - Get unread
TOOL: rss_search_articles - Search articles
TOOL: rss_get_stats - Get RSS stats

## STORY CLUSTERING (8)

TOOL: story_cluster_articles - Cluster articles
TOOL: story_get_cluster - Get cluster
TOOL: story_list_clusters - List clusters
TOOL: story_merge_clusters - Merge clusters
TOOL: story_split_cluster - Split cluster
TOOL: story_add_article - Add to cluster
TOOL: story_remove_article - Remove from cluster
TOOL: story_get_timeline - Get timeline

## BIAS ANALYSIS (11)

TOOL: analysis_bias_check - Check bias
TOOL: analysis_framing_detect - Detect framing
TOOL: analysis_sentiment - Analyze sentiment
TOOL: analysis_fact_check - Fact check
TOOL: analysis_source_credibility - Check credibility
TOOL: analysis_narrative_compare - Compare narratives
TOOL: analysis_get_indicators - Get indicators
TOOL: analysis_batch_process - Batch analyze
TOOL: analysis_get_history - Get history
TOOL: analysis_set_thresholds - Set thresholds
TOOL: analysis_get_stats - Get analysis stats

## SCRIPT GENERATION (20)

TOOL: script_generate - Generate script
TOOL: script_get - Get script
TOOL: script_list - List scripts
TOOL: script_update - Update script
TOOL: script_delete - Delete script
TOOL: script_set_template - Set template
TOOL: script_get_templates - Get templates
TOOL: script_apply_style - Apply style
TOOL: script_add_section - Add section
TOOL: script_remove_section - Remove section
TOOL: script_reorder - Reorder sections
TOOL: script_calculate_timing - Calculate timing
TOOL: script_validate - Validate script
TOOL: script_convert_format - Convert format
TOOL: script_merge - Merge scripts
TOOL: script_split - Split script
TOOL: script_add_transition - Add transition
TOOL: script_set_pacing - Set pacing
TOOL: script_learn_from_edit - Learn from edit
TOOL: script_get_suggestions - Get suggestions

## NOTION INTEGRATION (5)

TOOL: notion_sync_stories - Sync to Notion
TOOL: notion_get_pending - Get pending
TOOL: notion_approve_story - Approve story
TOOL: notion_reject_story - Reject story
TOOL: notion_get_approved - Get approved

## TELEPROMPTER EXPORT (9)

TOOL: teleprompter_export - Export script
TOOL: teleprompter_format - Format script
TOOL: teleprompter_set_speed - Set speed
TOOL: teleprompter_preview - Preview
TOOL: teleprompter_airdrop - AirDrop to iPad (NOTES: Requires macOS with AirDrop enabled)
TOOL: teleprompter_queue_add - Add to queue
TOOL: teleprompter_queue_list - List queue
TOOL: teleprompter_get_device_status - Device status
TOOL: teleprompter_sync - Sync to device

## VIDEO PRODUCTION (14)

TOOL: video_generate_background - Generate background
TOOL: video_composite - Composite layers
TOOL: video_add_overlay - Add overlay
TOOL: video_add_lower_third - Add lower third
TOOL: video_add_transition - Add transition
TOOL: video_set_resolution - Set resolution
TOOL: video_set_format - Set format
TOOL: video_render - Render video
TOOL: video_preview - Preview video
TOOL: video_export - Export video
TOOL: video_get_templates - Get templates
TOOL: video_apply_template - Apply template
TOOL: video_batch_render - Batch render
TOOL: video_get_render_status - Render status

## ORCHESTRATOR (8)

TOOL: orchestrator_run_overnight - Run overnight
TOOL: orchestrator_get_schedule - Get schedule
TOOL: orchestrator_add_task - Add task
TOOL: orchestrator_remove_task - Remove task
TOOL: orchestrator_get_status - Get status
TOOL: orchestrator_pause - Pause
TOOL: orchestrator_resume - Resume
TOOL: orchestrator_get_logs - Get logs

---

KEY CONCEPTS

OUTLET: News source with bias and credibility ratings
FEED: RSS feed attached to an outlet
CLUSTER: Group of related articles about same story
SCRIPT: Teleprompter-ready news script
TREATMENT: Story presentation approach (breaking, feature, etc.)

---

WORKFLOW

1. INGEST: RSS feeds polled on schedule
2. CLUSTER: Articles grouped by story/topic
3. ANALYZE: Bias check, framing detection, fact check
4. SCRIPT: Generate teleprompter scripts
5. APPROVE: Notion workflow for editorial review
6. PRODUCE: Video background generation
7. DELIVER: Teleprompter export, AirDrop to iPad

---

DATABASE SCHEMA

TABLE: outlets - id, name, url, type, bias, credibility, region, language
TABLE: feeds - id, outlet_id, url, category, poll_interval, last_polled, status
TABLE: articles - id, feed_id, outlet_id, url, title, content, published_at, fetched_at, read
TABLE: clusters - id, topic, articles, similarity_score, status, created_at, updated_at
TABLE: scripts - id, cluster_id, content, format, status, version, created_at, updated_at
TABLE: videos - id, script_id, style, resolution, format, status, path, created_at
TABLE: analysis_results - id, article_id, type, results, created_at

---

INTEGRATIONS

PERPLEXITY: Direct API for research queries
COMET_FLOW: Workflow engine for complex research
NOTION: Story approval workflow
AIRDROP: iPad delivery for teleprompter
LAUNCHD: macOS scheduling for overnight operations
CHRONOS_SYNAPSE: Event tracking

---

KEY FILES

SOURCE: /repo/niws-server/
INDEX: src/index.ts
DATABASE: src/database.ts
OUTLETS: src/outlets/*.ts
RSS: src/rss/*.ts
ANALYSIS: src/analysis/*.ts
SCRIPTS: src/scripts/*.ts
NOTION: src/notion/*.ts
TELEPROMPTER: src/teleprompter/*.ts
VIDEO: src/video/*.ts
ORCHESTRATOR: src/orchestrator/*.ts
STRATEGY: src/strategy/*.ts
TYPES: src/types.ts
DATA: data/niws.db
CONFIG: src/config/
