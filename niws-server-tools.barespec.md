TOOLS: niws-server
VERSION: 1.0
UPDATED: 2025-12-25
COUNT: 102

---

## CORE RESEARCH (4 tools)

---

TOOL: research_query
STATUS: ✅ Implemented
INPUT: { query: string (required), backend?: "auto"|"direct_perplexity"|"comet_flow" (default: "auto"), flow_id?: string, model?: "sonar-small"|"sonar"|"sonar-pro" (default: "sonar-small"), maxTokens?: number (default: 1000), importance?: "low"|"medium"|"high" (default: "medium"), searchDomainFilter?: string[], searchRecencyFilter?: "day"|"week"|"month"|"year", returnCitations?: boolean (default: true) }
OUTPUT: { answer: string, citations: array, backend_used: string, tokens_used: number, quality_score: number, cached: boolean }
USE: Execute research via best available backend
EXAMPLE: research_query({ query: "latest AI agent developments 2025", importance: "high" })
NOTES: "auto" selects backend based on query complexity and budget.

---

TOOL: get_research_stats
STATUS: ✅ Implemented
INPUT: { time_range?: "hour"|"day"|"week"|"month" }
OUTPUT: { queries_total: number, backend_usage: object, avg_quality_score: number, cache_hit_rate: number, budget_used: number }
USE: Get research query statistics
EXAMPLE: get_research_stats({ time_range: "week" })

---

TOOL: get_budget_status
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { daily_budget: number, used_today: number, remaining: number, reset_time: timestamp }
USE: Get current budget status
EXAMPLE: get_budget_status({})

---

TOOL: get_cache_stats
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { total_cached: number, hit_rate: number, size_bytes: number, oldest_entry: timestamp }
USE: Get cache performance statistics
EXAMPLE: get_cache_stats({})

---

## OUTLET DATABASE (10 tools)

---

TOOL: outlet_add
STATUS: ✅ Implemented
INPUT: { name: string (required), url: string (required), type: "wire_service"|"newspaper"|"broadcast"|"digital"|"magazine" (required), bias?: "far_left"|"left"|"center_left"|"center"|"center_right"|"right"|"far_right", credibility?: number (1-10), region?: string, language?: string (default: "en") }
OUTPUT: { outlet_id: string, added: boolean }
USE: Add a news outlet to the database
EXAMPLE: outlet_add({ name: "Reuters", url: "https://reuters.com", type: "wire_service", bias: "center", credibility: 9 })

---

TOOL: outlet_search
STATUS: ✅ Implemented
INPUT: { query?: string, type?: string, bias?: string, min_credibility?: number, limit?: number (default: 20) }
OUTPUT: { outlets: array, total: number }
USE: Search news outlets
EXAMPLE: outlet_search({ type: "newspaper", bias: "center", min_credibility: 7 })

---

TOOL: outlet_get
STATUS: ✅ Implemented
INPUT: { outlet_id: string (required) }
OUTPUT: { outlet: object, feeds: array, article_count: number }
USE: Get outlet details with associated feeds
EXAMPLE: outlet_get({ outlet_id: "out_reuters" })

---

TOOL: outlet_update
STATUS: ✅ Implemented
INPUT: { outlet_id: string (required), updates: object (required) }
OUTPUT: { updated: boolean, outlet: object }
USE: Update outlet properties
EXAMPLE: outlet_update({ outlet_id: "out_123", updates: { credibility: 8 } })

---

TOOL: outlet_delete
STATUS: ✅ Implemented
INPUT: { outlet_id: string (required), confirm: boolean (required) }
OUTPUT: { deleted: boolean, feeds_removed: number }
USE: Delete outlet and associated feeds
EXAMPLE: outlet_delete({ outlet_id: "out_123", confirm: true })

---

TOOL: outlet_list_by_type
STATUS: ✅ Implemented
INPUT: { type: string (required), limit?: number }
OUTPUT: { outlets: array, total: number }
USE: List outlets by type
EXAMPLE: outlet_list_by_type({ type: "wire_service" })

---

TOOL: outlet_list_by_bias
STATUS: ✅ Implemented
INPUT: { bias: string (required), limit?: number }
OUTPUT: { outlets: array, total: number }
USE: List outlets by political bias
EXAMPLE: outlet_list_by_bias({ bias: "center" })

---

TOOL: outlet_get_stats
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { total_outlets: number, by_type: object, by_bias: object, avg_credibility: number }
USE: Get outlet database statistics
EXAMPLE: outlet_get_stats({})

---

TOOL: outlet_import
STATUS: ✅ Implemented
INPUT: { outlets: array (required), merge_strategy?: "replace"|"merge"|"skip" (default: "skip") }
OUTPUT: { imported: number, skipped: number, errors: array }
USE: Bulk import outlets
EXAMPLE: outlet_import({ outlets: [...], merge_strategy: "merge" })

---

TOOL: outlet_export
STATUS: ✅ Implemented
INPUT: { format?: "json"|"csv" (default: "json"), types?: string[] }
OUTPUT: { outlets: array, count: number, format: string }
USE: Export outlet database
EXAMPLE: outlet_export({ format: "json" })

---

## RSS BACKEND (13 tools)

---

TOOL: rss_add_feed
STATUS: ✅ Implemented
INPUT: { outlet_id: string (required), url: string (required), category?: string }
OUTPUT: { feed_id: string, added: boolean }
USE: Add an RSS feed for an outlet
EXAMPLE: rss_add_feed({ outlet_id: "out_reuters", url: "https://reuters.com/rss/world" })

---

TOOL: rss_remove_feed
STATUS: ✅ Implemented
INPUT: { feed_id: string (required) }
OUTPUT: { removed: boolean }
USE: Remove an RSS feed
EXAMPLE: rss_remove_feed({ feed_id: "feed_123" })

---

TOOL: rss_get_feed
STATUS: ✅ Implemented
INPUT: { feed_id: string (required) }
OUTPUT: { feed: object, last_polled: timestamp, article_count: number }
USE: Get feed details
EXAMPLE: rss_get_feed({ feed_id: "feed_123" })

---

TOOL: rss_list_feeds
STATUS: ✅ Implemented
INPUT: { outlet_id?: string, active_only?: boolean (default: true) }
OUTPUT: { feeds: array, total: number }
USE: List RSS feeds
EXAMPLE: rss_list_feeds({ outlet_id: "out_reuters" })

---

TOOL: rss_poll_feeds
STATUS: ✅ Implemented
INPUT: { outlet_ids?: string[], force?: boolean (default: false) }
OUTPUT: { articles_fetched: number, feeds_polled: number, errors: array }
USE: Poll RSS feeds for new articles
EXAMPLE: rss_poll_feeds({})
NOTES: Polls all feeds if outlet_ids not specified.

---

TOOL: rss_poll_single
STATUS: ✅ Implemented
INPUT: { feed_id: string (required) }
OUTPUT: { articles_fetched: number, new_articles: array }
USE: Poll a single feed
EXAMPLE: rss_poll_single({ feed_id: "feed_123" })

---

TOOL: rss_get_articles
STATUS: ✅ Implemented
INPUT: { feed_id?: string, outlet_id?: string, since?: timestamp, limit?: number (default: 50) }
OUTPUT: { articles: array, total: number }
USE: Get articles from feeds
EXAMPLE: rss_get_articles({ outlet_id: "out_reuters", limit: 20 })

---

TOOL: rss_set_poll_interval
STATUS: ✅ Implemented
INPUT: { feed_id: string (required), interval_minutes: number (required) }
OUTPUT: { updated: boolean, next_poll: timestamp }
USE: Set poll interval for a feed
EXAMPLE: rss_set_poll_interval({ feed_id: "feed_123", interval_minutes: 30 })

---

TOOL: rss_get_poll_status
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { active_feeds: number, last_poll: timestamp, next_poll: timestamp, errors: array }
USE: Get polling status
EXAMPLE: rss_get_poll_status({})

---

TOOL: rss_mark_article_read
STATUS: ✅ Implemented
INPUT: { article_id: string (required) }
OUTPUT: { marked: boolean }
USE: Mark article as read
EXAMPLE: rss_mark_article_read({ article_id: "art_123" })

---

TOOL: rss_get_unread
STATUS: ✅ Implemented
INPUT: { outlet_id?: string, limit?: number (default: 100) }
OUTPUT: { articles: array, total: number }
USE: Get unread articles
EXAMPLE: rss_get_unread({ limit: 50 })

---

TOOL: rss_search_articles
STATUS: ✅ Implemented
INPUT: { query: string (required), outlet_ids?: string[], since?: timestamp, limit?: number }
OUTPUT: { articles: array, total: number }
USE: Search article content
EXAMPLE: rss_search_articles({ query: "climate change", limit: 20 })

---

TOOL: rss_get_stats
STATUS: ✅ Implemented
INPUT: { period?: "day"|"week"|"month" (default: "day") }
OUTPUT: { articles_fetched: number, feeds_polled: number, avg_fetch_time: number, errors: number }
USE: Get RSS polling statistics
EXAMPLE: rss_get_stats({ period: "week" })

---

## STORY CLUSTERING (8 tools)

---

TOOL: story_cluster_articles
STATUS: ✅ Implemented
INPUT: { time_range?: string (default: "24h"), min_cluster_size?: number (default: 2), similarity_threshold?: number (default: 0.7) }
OUTPUT: { clusters: [{ cluster_id: string, topic: string, articles: array, similarity_score: number }], total_clusters: number }
USE: Cluster articles into story groups
EXAMPLE: story_cluster_articles({ min_cluster_size: 3 })

---

TOOL: story_get_cluster
STATUS: ✅ Implemented
INPUT: { cluster_id: string (required) }
OUTPUT: { cluster: object, articles: array, timeline: array, sources: array }
USE: Get cluster details
EXAMPLE: story_get_cluster({ cluster_id: "clust_123" })

---

TOOL: story_list_clusters
STATUS: ✅ Implemented
INPUT: { status?: "active"|"archived"|"all" (default: "active"), limit?: number (default: 20) }
OUTPUT: { clusters: array, total: number }
USE: List story clusters
EXAMPLE: story_list_clusters({ status: "active" })

---

TOOL: story_merge_clusters
STATUS: ✅ Implemented
INPUT: { cluster_ids: string[] (required), new_topic?: string }
OUTPUT: { merged_cluster_id: string, articles_combined: number }
USE: Merge related clusters
EXAMPLE: story_merge_clusters({ cluster_ids: ["clust_1", "clust_2"] })

---

TOOL: story_split_cluster
STATUS: ✅ Implemented
INPUT: { cluster_id: string (required), article_ids: string[] (required) }
OUTPUT: { new_cluster_id: string, articles_moved: number }
USE: Split articles into new cluster
EXAMPLE: story_split_cluster({ cluster_id: "clust_123", article_ids: ["art_1", "art_2"] })

---

TOOL: story_add_article
STATUS: ✅ Implemented
INPUT: { cluster_id: string (required), article_id: string (required) }
OUTPUT: { added: boolean, new_similarity: number }
USE: Add article to cluster
EXAMPLE: story_add_article({ cluster_id: "clust_123", article_id: "art_456" })

---

TOOL: story_remove_article
STATUS: ✅ Implemented
INPUT: { cluster_id: string (required), article_id: string (required) }
OUTPUT: { removed: boolean }
USE: Remove article from cluster
EXAMPLE: story_remove_article({ cluster_id: "clust_123", article_id: "art_456" })

---

TOOL: story_get_timeline
STATUS: ✅ Implemented
INPUT: { cluster_id: string (required) }
OUTPUT: { timeline: [{ timestamp: timestamp, event: string, article_id?: string }] }
USE: Get story development timeline
EXAMPLE: story_get_timeline({ cluster_id: "clust_123" })

---

## BIAS ANALYSIS (11 tools)

---

TOOL: analysis_bias_check
STATUS: ✅ Implemented
INPUT: { content: string (required), compare_to?: string[] }
OUTPUT: { bias_score: number (-1 to 1), lean: "far_left"|"left"|"center_left"|"center"|"center_right"|"right"|"far_right", confidence: number, indicators: array }
USE: Analyze content for political bias
EXAMPLE: analysis_bias_check({ content: "Article text..." })

---

TOOL: analysis_framing_detect
STATUS: ✅ Implemented
INPUT: { content: string (required) }
OUTPUT: { frames: [{ type: string, strength: number, examples: array }], dominant_frame: string }
USE: Detect narrative framing
EXAMPLE: analysis_framing_detect({ content: "Article text..." })

---

TOOL: analysis_sentiment
STATUS: ✅ Implemented
INPUT: { content: string (required), aspects?: string[] }
OUTPUT: { overall: number (-1 to 1), by_aspect: object, key_phrases: array }
USE: Analyze sentiment
EXAMPLE: analysis_sentiment({ content: "Article text...", aspects: ["economy", "policy"] })

---

TOOL: analysis_fact_check
STATUS: ✅ Implemented
INPUT: { claims: string[] (required) }
OUTPUT: { results: [{ claim: string, verdict: "true"|"false"|"mixed"|"unverified", confidence: number, sources: array }] }
USE: Fact-check claims
EXAMPLE: analysis_fact_check({ claims: ["GDP grew 3% last quarter"] })

---

TOOL: analysis_source_credibility
STATUS: ✅ Implemented
INPUT: { outlet_id?: string, url?: string }
OUTPUT: { credibility_score: number, factors: object, warnings: array }
USE: Check source credibility
EXAMPLE: analysis_source_credibility({ outlet_id: "out_123" })

---

TOOL: analysis_narrative_compare
STATUS: ✅ Implemented
INPUT: { article_ids: string[] (required) }
OUTPUT: { narratives: array, divergence_points: array, common_facts: array }
USE: Compare narratives across articles
EXAMPLE: analysis_narrative_compare({ article_ids: ["art_1", "art_2", "art_3"] })

---

TOOL: analysis_get_indicators
STATUS: ✅ Implemented
INPUT: { type?: "bias"|"framing"|"sentiment"|"all" }
OUTPUT: { indicators: [{ name: string, description: string, weight: number }] }
USE: Get analysis indicators used
EXAMPLE: analysis_get_indicators({ type: "bias" })

---

TOOL: analysis_batch_process
STATUS: ✅ Implemented
INPUT: { article_ids: string[] (required), analyses: ("bias"|"framing"|"sentiment")[] (required) }
OUTPUT: { results: object, processed: number, errors: array }
USE: Batch analyze articles
EXAMPLE: analysis_batch_process({ article_ids: [...], analyses: ["bias", "sentiment"] })

---

TOOL: analysis_get_history
STATUS: ✅ Implemented
INPUT: { article_id?: string, outlet_id?: string, limit?: number }
OUTPUT: { analyses: array, total: number }
USE: Get analysis history
EXAMPLE: analysis_get_history({ outlet_id: "out_123", limit: 10 })

---

TOOL: analysis_set_thresholds
STATUS: ✅ Implemented
INPUT: { thresholds: object (required) }
OUTPUT: { updated: boolean, new_thresholds: object }
USE: Set analysis thresholds
EXAMPLE: analysis_set_thresholds({ thresholds: { bias_warning: 0.5 } })

---

TOOL: analysis_get_stats
STATUS: ✅ Implemented
INPUT: { period?: "day"|"week"|"month" }
OUTPUT: { articles_analyzed: number, avg_bias: number, bias_distribution: object }
USE: Get analysis statistics
EXAMPLE: analysis_get_stats({ period: "week" })

---

## SCRIPT GENERATION (20 tools)

---

TOOL: script_generate
STATUS: ✅ Implemented
INPUT: { story_id: string (required), format?: "teleprompter"|"social"|"blog" (default: "teleprompter"), template_id?: string, tone?: "neutral"|"urgent"|"casual", target_length?: number }
OUTPUT: { script_id: string, script: string, word_count: number, reading_time: number, format: string }
USE: Generate script from story cluster
EXAMPLE: script_generate({ story_id: "story_123", format: "teleprompter" })

---

TOOL: script_get
STATUS: ✅ Implemented
INPUT: { script_id: string (required) }
OUTPUT: { script: object, versions: array, status: string }
USE: Get script details
EXAMPLE: script_get({ script_id: "scr_123" })

---

TOOL: script_list
STATUS: ✅ Implemented
INPUT: { status?: "draft"|"approved"|"archived", story_id?: string, limit?: number }
OUTPUT: { scripts: array, total: number }
USE: List scripts
EXAMPLE: script_list({ status: "draft" })

---

TOOL: script_update
STATUS: ✅ Implemented
INPUT: { script_id: string (required), content: string (required), note?: string }
OUTPUT: { updated: boolean, version: number }
USE: Update script content
EXAMPLE: script_update({ script_id: "scr_123", content: "Updated text...", note: "Fixed intro" })

---

TOOL: script_delete
STATUS: ✅ Implemented
INPUT: { script_id: string (required), confirm: boolean (required) }
OUTPUT: { deleted: boolean }
USE: Delete script
EXAMPLE: script_delete({ script_id: "scr_123", confirm: true })

---

TOOL: script_set_template
STATUS: ✅ Implemented
INPUT: { name: string (required), template: string (required), format: string (required) }
OUTPUT: { template_id: string, created: boolean }
USE: Create script template
EXAMPLE: script_set_template({ name: "breaking_news", template: "{{headline}}...", format: "teleprompter" })

---

TOOL: script_get_templates
STATUS: ✅ Implemented
INPUT: { format?: string }
OUTPUT: { templates: array }
USE: Get available templates
EXAMPLE: script_get_templates({ format: "teleprompter" })

---

TOOL: script_apply_style
STATUS: ✅ Implemented
INPUT: { script_id: string (required), style: object (required) }
OUTPUT: { styled: boolean, script: string }
USE: Apply styling to script
EXAMPLE: script_apply_style({ script_id: "scr_123", style: { emphasis: "bold_key_phrases" } })

---

TOOL: script_add_section
STATUS: ✅ Implemented
INPUT: { script_id: string (required), section: object (required), position?: number }
OUTPUT: { added: boolean, section_id: string }
USE: Add section to script
EXAMPLE: script_add_section({ script_id: "scr_123", section: { type: "transition", content: "..." } })

---

TOOL: script_remove_section
STATUS: ✅ Implemented
INPUT: { script_id: string (required), section_id: string (required) }
OUTPUT: { removed: boolean }
USE: Remove section from script
EXAMPLE: script_remove_section({ script_id: "scr_123", section_id: "sec_456" })

---

TOOL: script_reorder
STATUS: ✅ Implemented
INPUT: { script_id: string (required), section_order: string[] (required) }
OUTPUT: { reordered: boolean }
USE: Reorder script sections
EXAMPLE: script_reorder({ script_id: "scr_123", section_order: ["sec_1", "sec_3", "sec_2"] })

---

TOOL: script_calculate_timing
STATUS: ✅ Implemented
INPUT: { script_id: string (required), wpm?: number (default: 150) }
OUTPUT: { total_time: number, section_times: array, pacing_suggestions: array }
USE: Calculate reading timing
EXAMPLE: script_calculate_timing({ script_id: "scr_123", wpm: 160 })

---

TOOL: script_validate
STATUS: ✅ Implemented
INPUT: { script_id: string (required) }
OUTPUT: { valid: boolean, issues: array, suggestions: array }
USE: Validate script for production
EXAMPLE: script_validate({ script_id: "scr_123" })

---

TOOL: script_convert_format
STATUS: ✅ Implemented
INPUT: { script_id: string (required), target_format: "teleprompter"|"social"|"blog" (required) }
OUTPUT: { converted_script_id: string, format: string }
USE: Convert script to different format
EXAMPLE: script_convert_format({ script_id: "scr_123", target_format: "social" })

---

TOOL: script_merge
STATUS: ✅ Implemented
INPUT: { script_ids: string[] (required) }
OUTPUT: { merged_script_id: string, sections_combined: number }
USE: Merge multiple scripts
EXAMPLE: script_merge({ script_ids: ["scr_1", "scr_2"] })

---

TOOL: script_split
STATUS: ✅ Implemented
INPUT: { script_id: string (required), at_section: string (required) }
OUTPUT: { script_ids: string[] }
USE: Split script at section
EXAMPLE: script_split({ script_id: "scr_123", at_section: "sec_5" })

---

TOOL: script_add_transition
STATUS: ✅ Implemented
INPUT: { script_id: string (required), after_section: string (required), transition_type: string (required) }
OUTPUT: { added: boolean, transition_id: string }
USE: Add transition between sections
EXAMPLE: script_add_transition({ script_id: "scr_123", after_section: "sec_1", transition_type: "fade" })

---

TOOL: script_set_pacing
STATUS: ✅ Implemented
INPUT: { script_id: string (required), pacing: "slow"|"normal"|"fast" (required) }
OUTPUT: { updated: boolean, timing_adjustments: object }
USE: Set overall pacing
EXAMPLE: script_set_pacing({ script_id: "scr_123", pacing: "normal" })

---

TOOL: script_learn_from_edit
STATUS: ✅ Implemented
INPUT: { script_id: string (required), original: string (required), edited: string (required), category?: string }
OUTPUT: { learned: boolean, pattern_id?: string }
USE: Learn from manual edits
EXAMPLE: script_learn_from_edit({ script_id: "scr_123", original: "...", edited: "..." })

---

TOOL: script_get_suggestions
STATUS: ✅ Implemented
INPUT: { script_id: string (required) }
OUTPUT: { suggestions: [{ type: string, original: string, suggested: string, reason: string }] }
USE: Get improvement suggestions
EXAMPLE: script_get_suggestions({ script_id: "scr_123" })

---

## NOTION INTEGRATION (5 tools)

---

TOOL: notion_sync_stories
STATUS: ✅ Implemented
INPUT: { database_id: string (required), stories?: string[], status_filter?: string }
OUTPUT: { synced_count: number, errors: array }
USE: Sync stories to Notion database
EXAMPLE: notion_sync_stories({ database_id: "db_123" })

---

TOOL: notion_get_pending
STATUS: ✅ Implemented
INPUT: { database_id: string (required) }
OUTPUT: { stories: array, total: number }
USE: Get stories pending approval
EXAMPLE: notion_get_pending({ database_id: "db_123" })

---

TOOL: notion_approve_story
STATUS: ✅ Implemented
INPUT: { page_id: string (required), treatment?: string, notes?: string }
OUTPUT: { approved: boolean, story_id: string }
USE: Approve story for production
EXAMPLE: notion_approve_story({ page_id: "page_123", treatment: "breaking" })

---

TOOL: notion_reject_story
STATUS: ✅ Implemented
INPUT: { page_id: string (required), reason: string (required) }
OUTPUT: { rejected: boolean }
USE: Reject story
EXAMPLE: notion_reject_story({ page_id: "page_123", reason: "Not newsworthy" })

---

TOOL: notion_get_approved
STATUS: ✅ Implemented
INPUT: { database_id: string (required), since?: timestamp }
OUTPUT: { stories: array, total: number }
USE: Get approved stories for production
EXAMPLE: notion_get_approved({ database_id: "db_123" })

---

## TELEPROMPTER EXPORT (9 tools)

---

TOOL: teleprompter_export
STATUS: ✅ Implemented
INPUT: { script_id: string (required), format?: "txt"|"docx"|"pdf" (default: "txt") }
OUTPUT: { export_path: string, format: string, success: boolean }
USE: Export script for teleprompter
EXAMPLE: teleprompter_export({ script_id: "scr_123", format: "txt" })

---

TOOL: teleprompter_format
STATUS: ✅ Implemented
INPUT: { script_id: string (required), options?: { font_size?: number, line_spacing?: number, margins?: object } }
OUTPUT: { formatted: boolean, preview: string }
USE: Format for teleprompter display
EXAMPLE: teleprompter_format({ script_id: "scr_123", options: { font_size: 32 } })

---

TOOL: teleprompter_set_speed
STATUS: ✅ Implemented
INPUT: { script_id: string (required), wpm: number (required) }
OUTPUT: { updated: boolean, total_duration: number }
USE: Set reading speed
EXAMPLE: teleprompter_set_speed({ script_id: "scr_123", wpm: 150 })

---

TOOL: teleprompter_preview
STATUS: ✅ Implemented
INPUT: { script_id: string (required) }
OUTPUT: { preview_url: string, expires: timestamp }
USE: Generate preview URL
EXAMPLE: teleprompter_preview({ script_id: "scr_123" })

---

TOOL: teleprompter_airdrop
STATUS: ✅ Implemented
INPUT: { script_id: string (required), device_name?: string }
OUTPUT: { sent: boolean, device: string }
USE: Send to iPad via AirDrop
EXAMPLE: teleprompter_airdrop({ script_id: "scr_123" })
NOTES: Requires macOS with AirDrop enabled.

---

TOOL: teleprompter_queue_add
STATUS: ✅ Implemented
INPUT: { script_id: string (required), position?: number }
OUTPUT: { added: boolean, queue_position: number }
USE: Add to teleprompter queue
EXAMPLE: teleprompter_queue_add({ script_id: "scr_123" })

---

TOOL: teleprompter_queue_list
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { queue: array, total: number }
USE: List teleprompter queue
EXAMPLE: teleprompter_queue_list({})

---

TOOL: teleprompter_get_device_status
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { devices: [{ name: string, connected: boolean, last_seen: timestamp }] }
USE: Check connected devices
EXAMPLE: teleprompter_get_device_status({})

---

TOOL: teleprompter_sync
STATUS: ✅ Implemented
INPUT: { device_name: string (required) }
OUTPUT: { synced: boolean, scripts_sent: number }
USE: Sync queue to device
EXAMPLE: teleprompter_sync({ device_name: "iPad Pro" })

---

## VIDEO PRODUCTION (14 tools)

---

TOOL: video_generate_background
STATUS: ✅ Implemented
INPUT: { story_id: string (required), style?: "abstract"|"news"|"minimal"|"dramatic", duration?: number }
OUTPUT: { video_path: string, duration: number, style: string }
USE: Generate video background
EXAMPLE: video_generate_background({ story_id: "story_123", style: "news" })

---

TOOL: video_composite
STATUS: ✅ Implemented
INPUT: { background_id: string (required), overlay_ids?: string[], output_format?: string }
OUTPUT: { video_id: string, path: string, duration: number }
USE: Composite video layers
EXAMPLE: video_composite({ background_id: "bg_123", overlay_ids: ["ov_1", "ov_2"] })

---

TOOL: video_add_overlay
STATUS: ✅ Implemented
INPUT: { video_id: string (required), overlay: object (required), position?: object, timing?: object }
OUTPUT: { overlay_id: string, added: boolean }
USE: Add overlay to video
EXAMPLE: video_add_overlay({ video_id: "vid_123", overlay: { type: "text", content: "Breaking News" } })

---

TOOL: video_add_lower_third
STATUS: ✅ Implemented
INPUT: { video_id: string (required), text: string (required), subtitle?: string, duration?: number, start_time?: number }
OUTPUT: { lower_third_id: string, added: boolean }
USE: Add lower third graphic
EXAMPLE: video_add_lower_third({ video_id: "vid_123", text: "John Smith", subtitle: "Correspondent" })

---

TOOL: video_add_transition
STATUS: ✅ Implemented
INPUT: { video_id: string (required), type: "fade"|"cut"|"dissolve"|"wipe", at_time: number (required), duration?: number }
OUTPUT: { transition_id: string, added: boolean }
USE: Add transition
EXAMPLE: video_add_transition({ video_id: "vid_123", type: "dissolve", at_time: 5.0 })

---

TOOL: video_set_resolution
STATUS: ✅ Implemented
INPUT: { video_id: string (required), resolution: "720p"|"1080p"|"4k" (required) }
OUTPUT: { updated: boolean, new_resolution: string }
USE: Set output resolution
EXAMPLE: video_set_resolution({ video_id: "vid_123", resolution: "1080p" })

---

TOOL: video_set_format
STATUS: ✅ Implemented
INPUT: { video_id: string (required), format: "mp4"|"mov"|"webm" (required) }
OUTPUT: { updated: boolean, new_format: string }
USE: Set output format
EXAMPLE: video_set_format({ video_id: "vid_123", format: "mp4" })

---

TOOL: video_render
STATUS: ✅ Implemented
INPUT: { video_id: string (required), quality?: "draft"|"standard"|"high" (default: "standard") }
OUTPUT: { render_id: string, status: "queued"|"rendering"|"complete", estimated_time: number }
USE: Render final video
EXAMPLE: video_render({ video_id: "vid_123", quality: "high" })

---

TOOL: video_preview
STATUS: ✅ Implemented
INPUT: { video_id: string (required) }
OUTPUT: { preview_url: string, expires: timestamp }
USE: Generate preview URL
EXAMPLE: video_preview({ video_id: "vid_123" })

---

TOOL: video_export
STATUS: ✅ Implemented
INPUT: { video_id: string (required), destination?: string }
OUTPUT: { export_path: string, size_bytes: number, success: boolean }
USE: Export rendered video
EXAMPLE: video_export({ video_id: "vid_123", destination: "/exports/" })

---

TOOL: video_get_templates
STATUS: ✅ Implemented
INPUT: { category?: string }
OUTPUT: { templates: array }
USE: Get video templates
EXAMPLE: video_get_templates({ category: "news" })

---

TOOL: video_apply_template
STATUS: ✅ Implemented
INPUT: { video_id: string (required), template_id: string (required) }
OUTPUT: { applied: boolean }
USE: Apply template to video
EXAMPLE: video_apply_template({ video_id: "vid_123", template_id: "tpl_breaking_news" })

---

TOOL: video_batch_render
STATUS: ✅ Implemented
INPUT: { video_ids: string[] (required), quality?: string }
OUTPUT: { batch_id: string, queued: number, estimated_total_time: number }
USE: Batch render videos
EXAMPLE: video_batch_render({ video_ids: ["vid_1", "vid_2", "vid_3"] })

---

TOOL: video_get_render_status
STATUS: ✅ Implemented
INPUT: { render_id?: string, batch_id?: string }
OUTPUT: { status: string, progress: number, errors?: array }
USE: Check render status
EXAMPLE: video_get_render_status({ render_id: "rnd_123" })

---

## ORCHESTRATOR (8 tools)

---

TOOL: orchestrator_run_overnight
STATUS: ✅ Implemented
INPUT: { tasks?: string[], skip?: string[] }
OUTPUT: { execution_id: string, tasks_queued: number, estimated_duration: number }
USE: Run overnight orchestration
EXAMPLE: orchestrator_run_overnight({ tasks: ["poll_feeds", "cluster_stories", "analyze_bias"] })

---

TOOL: orchestrator_get_schedule
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { schedule: [{ task: string, cron: string, next_run: timestamp, enabled: boolean }] }
USE: Get orchestration schedule
EXAMPLE: orchestrator_get_schedule({})

---

TOOL: orchestrator_add_task
STATUS: ✅ Implemented
INPUT: { task: string (required), cron: string (required), params?: object }
OUTPUT: { task_id: string, added: boolean, next_run: timestamp }
USE: Add scheduled task
EXAMPLE: orchestrator_add_task({ task: "poll_feeds", cron: "0 */6 * * *" })

---

TOOL: orchestrator_remove_task
STATUS: ✅ Implemented
INPUT: { task_id: string (required) }
OUTPUT: { removed: boolean }
USE: Remove scheduled task
EXAMPLE: orchestrator_remove_task({ task_id: "task_123" })

---

TOOL: orchestrator_get_status
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { running: boolean, current_task?: string, queue: array, last_run: timestamp }
USE: Get orchestrator status
EXAMPLE: orchestrator_get_status({})

---

TOOL: orchestrator_pause
STATUS: ✅ Implemented
INPUT: { duration?: number }
OUTPUT: { paused: boolean, resume_at?: timestamp }
USE: Pause orchestration
EXAMPLE: orchestrator_pause({ duration: 3600000 })

---

TOOL: orchestrator_resume
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { resumed: boolean }
USE: Resume orchestration
EXAMPLE: orchestrator_resume({})

---

TOOL: orchestrator_get_logs
STATUS: ✅ Implemented
INPUT: { task_id?: string, execution_id?: string, limit?: number (default: 100) }
OUTPUT: { logs: array, total: number }
USE: Get orchestration logs
EXAMPLE: orchestrator_get_logs({ limit: 50 })

---

## DATABASE SCHEMA

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
