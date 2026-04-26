# skillsmp-search

A Claude Code skill for searching and installing community skills from [SkillsMP](https://skillsmp.com).

## What it does

- **Keyword search** — search by skill name, category, or exact terms
- **AI-powered search** — describe what you need in natural language (requires API key)
- **Install** — install skills directly from GitHub with `/plugin`

## Prerequisites

SkillsMP is free to search without an API key (50 requests/day, keyword only). For AI search and higher rate limits (500 requests/day), get an API key at [skillsmp.com/docs/api#authentication](https://skillsmp.com/docs/api#authentication).

The key is stored in `~/.claude/skillsmp-key` (permissions `600`).

## Usage

Ask Claude Code things like:

- "Search SkillsMP for a database migration skill"
- "Find me a skill for generating changelogs on skillsmp"
- "Install the top-rated Python linting skill from SkillsMP"
