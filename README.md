# news-digest

A configurable [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that automatically collects the latest news from multiple sources and generates a structured daily digest.

**What is Claude Code?** It's an AI coding assistant that runs in your terminal (or browser at [claude.ai/code](https://claude.ai/code)). You talk to it in plain language, and it can search the web, read files, write code, and more. This skill extends Claude Code with a news digest capability. [Learn more](https://docs.anthropic.com/en/docs/claude-code/overview).

**What is a skill?** A skill is a set of instructions that teaches Claude Code how to do a specific task. When you install this skill, Claude Code learns how to collect news and build a digest for you. [Learn more about skills](https://docs.anthropic.com/en/docs/claude-code/skills).

## What it does

You tell Claude Code your topics of interest (e.g. "AI", "Crypto", "Ukraine"), and it:

1. **Searches the web** for recent news on each topic
2. **Finds and parses RSS feeds** from relevant news sites (automatically)
3. **Visits websites** directly to catch the freshest headlines
4. **Deduplicates and ranks** stories by importance
5. **Generates a formatted digest** as a Markdown or HTML file

All of this happens automatically — you just say "run news digest" and get a ready-to-read report.

## Features

- **Multi-source collection** — web search, RSS feeds (auto-discovered), direct website scraping
- **Easy configuration** — just list your topics; everything else has sensible defaults
- **Two output formats** — clean Markdown tables or a styled dark-theme HTML page
- **Parallel collection** — each topic is processed simultaneously for speed
- **No dependencies** — uses only built-in Claude Code tools, nothing to install
- **Works everywhere** — in the terminal, on [claude.ai/code](https://claude.ai/code), or as a scheduled cloud agent

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) installed and working (CLI, desktop app, or web)
- An active Claude subscription (Pro, Max, or Team)

## Installation

**For all your projects (personal skill):**

```bash
git clone https://github.com/Garadons/claude-code-news-digest-skill.git ~/.claude/skills/news-digest
```

**For a specific project only:**

```bash
cd your-project
git clone https://github.com/Garadons/claude-code-news-digest-skill.git .claude/skills/news-digest
```

After cloning, restart Claude Code (or open a new session). The skill will be automatically recognized. You can verify by running `/skills` in Claude Code.

[Learn more about installing skills](https://code.claude.com/docs/en/skills).

## Project structure

```
news-digest/
├── SKILL.md                    # Instructions that Claude Code follows
├── news-digest.config.yml      # Your configuration (topics, language, format)
├── templates/
│   ├── digest.md               # Markdown output template
│   └── digest.html             # HTML output template (dark theme)
├── LICENSE
└── README.md
```

## Configuration

Edit `news-digest.config.yml` to set your preferences. The file has detailed comments explaining every option.

### Minimal config (just topics)

The only thing you need to specify is what topics you care about:

```yaml
topics:
  - name: "AI / LLM"
  - name: "Crypto"
```

Claude Code will automatically find relevant news sources, RSS feeds, and search queries for each topic.

### Adding preferred websites

If you have favorite news sites for a topic, list them — Claude Code will prioritize them and auto-discover their RSS feeds:

```yaml
topics:
  - name: "AI / LLM"
    websites:
      - https://techcrunch.com
      - https://openai.com/blog

  - name: "Crypto"
    websites:
      - https://coindesk.com
      - https://cointelegraph.com
```

### Changing the language

```yaml
# Report will be written in Ukrainian
language: uk
```

### Changing the output format

```yaml
file_output:
  # "md" for Markdown, "html" for styled dark-theme page
  file_format: html
```

### All available options

| Option | What it does | Default |
|--------|-------------|---------|
| `language` | Language of the digest | `en` |
| `digest_title` | Title shown at the top | `"News Digest"` |
| `max_items_per_topic` | Max news items per topic | `5` |
| `collection.rss` | Auto-discover and parse RSS feeds | `true` |
| `collection.websites` | Visit websites directly | `true` |
| `collection.web_search` | Search the web for news | `true` |
| `output` | Where to show results: `terminal`, `file`, or both | `[terminal]` |
| `file_output.directory` | Where to save files | `~/news-digests` |
| `file_output.file_format` | File format: `md` or `html` | `md` |
| `item_format.show_importance` | Show importance indicators | `true` |
| `item_format.show_source` | Show source name | `true` |
| `item_format.show_link` | Show link to original article | `true` |
| `item_format.description_length` | `short`, `medium`, or `detailed` | `short` |

Per-topic overrides: each topic can set its own `max_items` and `websites`.

## Usage

### Manual run

Open Claude Code and type:

```
run news digest
```

or any natural phrasing like:

```
get me today's news
what's new in AI and crypto?
give me a daily briefing
```

### Recurring (local)

To run the digest automatically every 8 hours while your terminal is open:

```
/loop 8h /news-digest
```

### Scheduled (cloud, fully automatic)

This is the most powerful option — the digest runs automatically on a schedule, even when your computer is off. It uses [Claude Code Scheduled Remote Agents](https://docs.anthropic.com/en/docs/claude-code/scheduled-tasks).

To set it up, ask Claude Code:

```
set up a scheduled agent to run news digest every morning at 9am
```

Claude Code will create a cloud trigger that runs this skill on your schedule.

## Output formats

### Markdown (`file_format: md`)

Table-based layout with emoji section headers. Readable in any Markdown viewer, GitHub, VS Code, etc.

### HTML (`file_format: html`)

Styled dark-theme page with:
- Editorial typography (Playfair Display + JetBrains Mono)
- Color-coded importance indicators (red/amber/green)
- Hover effects and entrance animations
- Responsive design for mobile
- Opens in any browser

### What a digest looks like

Each news item includes:
- Importance level (🔴 High / 🟡 Medium / 🟢 Low) — based on how many sources report the same story
- Source name (TechCrunch, CoinDesk, etc.)
- Description (length configurable)
- Direct link to the original article

Output files are saved to `~/news-digests/` by default. Re-running on the same day creates a new file (`2026-04-25-2.html`) instead of overwriting.

## How it works (under the hood)

For each topic, Claude Code runs three collection stages in parallel:

1. **Web Search** — interprets your topic name, generates smart search queries, finds recent articles and discovers new sources
2. **RSS Feed Discovery** — finds RSS feeds for all relevant websites (both yours and discovered ones), parses them for structured article data
3. **Website Scraping** — visits news sites directly to catch headlines not yet in RSS or search indexes

Then it merges, deduplicates, ranks by importance, and formats everything according to your config and chosen template.

## License

MIT
