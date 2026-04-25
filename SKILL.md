---
name: news-digest
description: Use when the user asks for a news digest, daily briefing, news summary, or wants to collect and summarize recent news across configured topics. Triggers on requests like "get me today's news", "run news digest", "what's happening in AI/crypto/etc today".
---

# News Digest

Collect recent news from multiple sources and generate a structured digest based on user configuration.

## Step 1 — Read Configuration

1. Read `news-digest.config.yml` from the same directory as this SKILL.md file.
2. Apply these defaults for any missing or `null` fields:
   - `language`: `en`
   - `digest_title`: `"News Digest"`
   - `max_items_per_topic`: `5`
   - `collection.rss`: `true`
   - `collection.websites`: `true`
   - `collection.web_search`: `true`
   - `output`: `[terminal]`
   - `file_output.directory`: `~/news-digests`
   - `file_output.filename_format`: `{date}.md`
   - `item_format.show_importance`: `true`
   - `item_format.show_source`: `true`
   - `item_format.show_link`: `true`
   - `item_format.description_length`: `short`

## Step 2 — Collect News

Process each topic from `topics` list. For each topic, run three collection stages in order.

### 2a. Web Search (if `collection.web_search: true`)

**Goal:** Discover the news landscape and find relevant sources.

1. **Interpret the topic.** Analyze the topic `name` and `websites` (if provided) to understand the actual subject area. Do NOT blindly use the topic name as a search query. Examples:
   - `name: "AI / LLM"` → subject: artificial intelligence, large language models, machine learning
   - `name: "Крипта"` → subject: cryptocurrency, bitcoin, ethereum, web3
   - `name: "My Tech Stuff"` (vague name, but `websites: [techcrunch.com, theverge.com]`) → subject: technology news, gadgets, software

2. **Generate 3-5 diverse search queries** based on the interpreted subject:
   - Breaking/recent news: `"{subject}" breaking news {current_month} {current_year}`
   - Industry-specific: `"{subject}" announcements releases updates this week`
   - Analysis/trends: `"{subject}" latest developments {current_year}`
   - Sub-topic specific (e.g. for AI: `"new AI model release this week"`, `"LLM benchmark results {current_month}"`)

3. **Run `WebSearch`** for each query.

4. **Collect results** — save title, snippet, URL for each result.

5. **Identify new sources** — note domains that appear frequently in results. These will be used in step 2b for RSS discovery.

### 2b. RSS Feed Discovery & Parsing (if `collection.rss: true`)

**Goal:** Get structured article data from RSS feeds.

Discover and parse RSS feeds for **all relevant websites** — both user-configured in `websites` and domains discovered via web search in step 2a.

For each website:

1. **Find its RSS feed** using one of these methods (try in order, stop at first success):
   a. `WebSearch` for `"{domain}" RSS feed URL` — often the fastest way to find the feed
   b. Try common RSS paths by appending to the domain: `/feed`, `/rss`, `/rss.xml`, `/feed.xml`, `/atom.xml`
   c. `WebFetch` the website's homepage HTML and look for `<link>` tags with `type="application/rss+xml"` or `type="application/atom+xml"`

2. **Fetch and parse the RSS feed** — `WebFetch` the discovered RSS URL. From the XML, extract for each item:
   - `title` — article title
   - `link` — article URL
   - `pubDate` or `updated` — publication date
   - `description` or `summary` — article description/snippet

3. **Filter to recent items only** — keep only items from the last 24-48 hours.

Do NOT spend too long on any single website. If RSS discovery fails after trying all three methods, skip that website and move on.

### 2c. Direct Website Scraping (if `collection.websites: true`)

**Goal:** Catch news not yet in RSS or search indexes.

For **all relevant websites** — both user-configured and discovered via web search in step 2a:

1. `WebFetch` the main page or relevant news section.
2. Extract headlines, dates, and links from the page content.
3. This catches very recent news that may not yet appear in RSS feeds or search results.

## Step 3 — Process Results

For each topic:

1. **Merge** all results from the three collection stages (2a, 2b, 2c).
2. **Deduplicate** — remove items pointing to the same article. Match by URL or very similar titles.
3. **Rank by importance:**
   - How many sources mention the same story (cross-source confirmation)
   - Recency (newer = more important)
   - Source authority (well-known outlets rank higher)
4. **Assign importance level** to each item:
   - 🔴 High — mentioned by 3+ sources, or breaking/major event
   - 🟡 Medium — mentioned by 2 sources, or notable development
   - 🟢 Low — single source, or minor update
5. **Trim** to `max_items` (use topic-level override if set, otherwise `max_items_per_topic`).

## Step 4 — Format Output

1. Read the template from `templates/digest.md`.
2. Build the digest following the template structure:
   - Replace `{digest_title}` with config value
   - Replace `{date}` with today's date (YYYY-MM-DD)
   - Replace `{timestamp}` with current time (HH:MM UTC)
   - Replace `{language}` with config value
   - Replace `{topic_count}` with number of topics
3. For each topic, create a `## {topic_name}` section.
4. For each news item in the topic, create a `### {article_title}` block:
   - Include/exclude fields based on `item_format` config (`show_importance`, `show_source`, `show_link`)
   - Description length follows `description_length` setting:
     - `short` — 1-2 sentences
     - `medium` — a paragraph (3-5 sentences)
     - `detailed` — extended analysis (2-3 paragraphs)
5. Write the entire digest in the language specified by `language` config.

## Step 5 — Deliver

Based on `output` config:

- **`terminal`** — Print the full formatted digest directly to the console.
- **`file`** — Create the output directory if it doesn't exist. Write the digest as a markdown file:
  - Directory: `file_output.directory` (expand `~` to home directory)
  - Filename: `file_output.filename_format` with `{date}` replaced by today's date (YYYY-MM-DD)
  - If the file already exists, overwrite it (re-running the digest for the same day updates it).

Report what was done: how many topics processed, how many total items, where the file was saved (if applicable).
