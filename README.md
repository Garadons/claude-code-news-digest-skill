# news-digest

A configurable Claude Code skill that collects news from multiple sources and generates a structured daily digest.

## Features

- **Multi-source collection** — RSS feeds (auto-discovered), direct website scraping, web search
- **Easy configuration** — just specify topics and optionally websites; everything else has sensible defaults
- **Configurable output** — terminal, file (Markdown or HTML), or both
- **Two output formats** — clean Markdown tables or styled dark-theme HTML page
- **No dependencies** — uses only built-in Claude Code tools (WebFetch, WebSearch)
- **Works everywhere** — locally (manual/loop) and in the cloud (Scheduled Remote Agent)

## Installation

Clone this repo into your Claude Code skills directory:

```bash
git clone https://github.com/USER/news-digest.git ~/.claude/skills/news-digest
```

Or add as a plugin source in Claude Code.

## Configuration

Edit `news-digest.config.yml` to set your preferences. The config file has detailed comments for every field.

### Minimal config

You only need to specify topics:

```yaml
language: uk

topics:
  - name: "AI / LLM"

  - name: "Crypto"
    websites:
      - https://coindesk.com
```

### Full config

See `news-digest.config.yml` for all available options including:
- Report language
- Collection methods (RSS, websites, web search)
- Output targets (terminal, file)
- File format (`md` or `html`)
- News item format (importance, source, description length)

## Usage

### Manual run

Ask Claude Code to run the digest:

```
> run news digest
> get me today's news
> what's new in AI and crypto today?
```

### Recurring (local)

```
/loop 8h /news-digest
```

### Scheduled (cloud)

Create a Scheduled Remote Agent trigger with this skill's repo as a source. The agent will run automatically on your schedule.

## Output

Two file formats available:

### Markdown (`file_format: md`)
Table-based layout with emoji section headers. Works in any markdown viewer.

### HTML (`file_format: html`)
Styled dark-theme page with editorial design — Playfair Display headers, color-coded importance indicators, hover effects, entrance animations. Opens in any browser.

Both formats include for each news item:
- Importance level (🔴 High / 🟡 Medium / 🟢 Low)
- Source name
- Description
- Link to original article

Output files are saved to `~/news-digests/` by default (configurable). Re-running on the same day creates a new file with a counter suffix (`-2`, `-3`, etc.) instead of overwriting.

## License

MIT
