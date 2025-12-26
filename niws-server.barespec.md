SERVER: niws-server
VERSION: 2.0
UPDATED: 2025-12-26
STATUS: Production
PORT: 8015 (HTTP only)
MCP: stdio transport (stdin/stdout JSON-RPC)
UDP: None (no InterLock mesh)
WS: None
DEPS: Notion API
PURPOSE: News Intelligence Workflow System - complete news production pipeline from RSS ingestion to video delivery
CONFIG: ENV ADMIN_HTTP_PORT (default 8015)

---

DOCUMENTATION STATUS

DOCUMENTED: 102 tools (fully documented in tools barespec)
NOTE: Complete documentation as of 2025-12-25.

---

ARCHITECTURE

FUNCTION: End-to-end news production pipeline
FLOW: RSS Ingestion → Clustering → Analysis → Scripting → Approval → Production → Delivery
STORAGE: SQLite (outlets, feeds, articles, clusters, scripts)
APPROVAL: Notion integration for story approval workflow
DELIVERY: Teleprompter export, AirDrop, video compositing

---

TOOL CATEGORIES

CORE_RESEARCH (4):
- research_query: Execute research via Perplexity or Comet
- get_research_stats: Query statistics
- get_budget_status: Budget tracking
- get_cache_stats: Cache performance

CITATION_VAULT (5):
- citation_add, citation_search, citation_get, citation_verify, citation_export

OUTLET_DATABASE (10):
- outlet_add, outlet_search, outlet_get, outlet_update, outlet_delete
- outlet_list_by_type, outlet_list_by_bias, outlet_get_stats
- outlet_import, outlet_export

RSS_BACKEND (13):
- rss_add_feed, rss_remove_feed, rss_get_feed, rss_list_feeds
- rss_poll_feeds, rss_poll_single, rss_get_articles
- rss_set_poll_interval, rss_get_poll_status
- rss_mark_article_read, rss_get_unread, rss_search_articles, rss_get_stats

STORY_CLUSTERING (8):
- story_cluster_articles, story_get_cluster, story_list_clusters
- story_merge_clusters, story_split_cluster, story_add_article
- story_remove_article, story_get_timeline

BIAS_ANALYSIS (11):
- analysis_bias_check, analysis_framing_detect, analysis_sentiment
- analysis_fact_check, analysis_source_credibility
- analysis_narrative_compare, analysis_get_indicators
- analysis_batch_process, analysis_get_history
- analysis_set_thresholds, analysis_get_stats

SCRIPT_GENERATION (20):
- script_generate, script_get, script_list, script_update, script_delete
- script_set_template, script_get_templates, script_apply_style
- script_add_section, script_remove_section, script_reorder
- script_calculate_timing, script_validate
- script_convert_format, script_merge, script_split
- script_add_transition, script_set_pacing
- script_learn_from_edit, script_get_suggestions

NOTION_INTEGRATION (5):
- notion_sync_stories, notion_get_pending, notion_approve_story
- notion_reject_story, notion_get_approved

TELEPROMPTER_EXPORT (9):
- teleprompter_export, teleprompter_format, teleprompter_set_speed
- teleprompter_preview, teleprompter_airdrop
- teleprompter_queue_add, teleprompter_queue_list
- teleprompter_get_device_status, teleprompter_sync

VIDEO_PRODUCTION (14):
- video_generate_background, video_composite, video_add_overlay
- video_add_lower_third, video_add_transition
- video_set_resolution, video_set_format
- video_render, video_preview, video_export
- video_get_templates, video_apply_template
- video_batch_render, video_get_render_status

ORCHESTRATOR (8):
- orchestrator_run_overnight, orchestrator_get_schedule
- orchestrator_add_task, orchestrator_remove_task
- orchestrator_get_status, orchestrator_pause, orchestrator_resume
- orchestrator_get_logs

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
