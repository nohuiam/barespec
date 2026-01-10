SERVER: niws-server
VERSION: 2.1
UPDATED: 2026-01-10
STATUS: DEPRECATED
PORT: 8015 (HTTP only)
MCP: stdio transport (stdin/stdout JSON-RPC)
UDP: None (no InterLock mesh)
WS: None
DEPS: Notion API, tenets-server (8027)
PURPOSE: News Intelligence Workflow System - complete news production pipeline from RSS ingestion to video delivery with Christ-Oh-Meter moral rating
CONFIG: ENV ADMIN_HTTP_PORT (default 8015)

---

DEPRECATION NOTICE

This monolithic server (26K LOC, 110 tools) has been decomposed into 4 specialized servers:

| Server | Ports | Tools | Purpose |
|--------|-------|-------|---------|
| niws-intake | UDP 3033, HTTP 8033, WS 9033 | 47 | RSS feeds, outlets, story clustering, bills |
| niws-analysis | UDP 3034, HTTP 8034, WS 9034 | 11 | Bias analysis, framing comparison |
| niws-production | UDP 3035, HTTP 8035, WS 9035 | 28 | Scripts, briefs, Christ-Oh-Meter |
| niws-delivery | UDP 3036, HTTP 8036, WS 9036 | 48 | Export, video, Notion, orchestration |

MIGRATION:
- New servers include InterLock mesh communication
- Databases preserved and migrated
- Tool names and APIs remain compatible
- See individual barespecs for complete documentation

BARESPECS:
- /repo/barespec/niws-intake.barespec.md
- /repo/barespec/niws-analysis.barespec.md
- /repo/barespec/niws-production.barespec.md
- /repo/barespec/niws-delivery.barespec.md

TIMELINE:
- 2026-01-10: Decomposition complete, new servers operational
- niws-server remains for reference only

---

---

DOCUMENTATION STATUS

DOCUMENTED: 110 tools (fully documented in tools barespec)
NOTE: Updated 2026-01-06 with Story Briefs module (8 tools).

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

STORY_BRIEFS (8):
- create_story_brief: Create structured brief from story cluster with quote/legislation extraction
- rate_christ_oh_meter: Rate action on Christ-Evil moral spectrum (-1.0 to +1.0) via tenets-server
- compare_quotes: Find quote variations across outlets for same speaker
- analyze_legislation: Extract factual, non-opinionated effects of legislation
- get_story_brief: Get story brief by ID
- list_story_briefs: List briefs with filters (status, moral_alignment, is_government_action)
- update_brief_status: Update workflow status (draft→reviewed→approved→used)
- get_brief_stats: Statistics on story briefs

---

KEY CONCEPTS

OUTLET: News source with bias and credibility ratings
FEED: RSS feed attached to an outlet
CLUSTER: Group of related articles about same story
SCRIPT: Teleprompter-ready news script
TREATMENT: Story presentation approach (breaking, feature, etc.)
STORY_BRIEF: Structured analysis with quotes, legislation, Christ-Oh-Meter rating
CHRIST_OH_METER: Moral rating spectrum from Tenets of Evil (-1.0) to Tenets of Christ (+1.0)

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
BRIEFS: src/briefs/*.ts (briefDatabase, christOhMeter, briefExtractor, briefTools, index)
STRATEGY: src/strategy/*.ts
TYPES: src/types.ts
DATA: data/niws.db, data/briefs.sqlite
CONFIG: src/config/

---

DATABASE SCHEMA

TABLE: outlets
COLUMNS: id, name, url, type, bias, credibility, region, language, created_at, updated_at
INDEXES: name, type, bias, credibility

TABLE: feeds
COLUMNS: id, outlet_id, url, category, poll_interval, last_polled, status
INDEXES: outlet_id, status

TABLE: articles
COLUMNS: id, feed_id, outlet_id, url, title, content, published_at, fetched_at, read
INDEXES: outlet_id, published_at, read

TABLE: clusters
COLUMNS: id, topic, articles, similarity_score, status, created_at, updated_at
INDEXES: status, created_at

TABLE: scripts
COLUMNS: id, cluster_id, content, format, status, version, created_at, updated_at
INDEXES: cluster_id, status, format

TABLE: videos
COLUMNS: id, script_id, style, resolution, format, status, path, created_at
INDEXES: script_id, status

TABLE: analysis_results
COLUMNS: id, article_id, type, results, created_at
INDEXES: article_id, type

TABLE: niws_story_briefs
COLUMNS: id, brief_id, story_cluster_id, headline, summary, topic_keywords (JSON)
         sources (JSON), outlet_count, lean_coverage (JSON)
         quotes (JSON), legislation (JSON), is_government_action, government_category
         christ_oh_meter (JSON), moral_alignment, moral_score
         verification_level, story_potential, created_at, updated_at
         status, used_in_episode
INDEXES: status, moral_alignment, is_government_action, created_at, story_cluster_id
