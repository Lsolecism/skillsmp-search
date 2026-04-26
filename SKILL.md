---
name: skillsmp-search
description: Search and install Claude Code skills from SkillsMP (skillsmp.com). Use this skill whenever the user mentions skillsmp, search-skillsmp, install-skill-from-skillsmp, or wants to find/install community skills from the SkillsMP marketplace. Also trigger when the user wants to search for a skill that isn't in the official Anthropic marketplace, or says things like "find me a skill for X on skillsmp" or "search the community skill market for X".
---

# SkillsMP Search & Install

Search for Claude Code skills on [SkillsMP](https://skillsmp.com) and install them from GitHub.

## Prerequisites

### API Key

SkillsMP requires an API key for full access (AI search, higher rate limits). Without a key, only keyword search is available with 50 requests/day.

The API key is stored in `~/.claude/skillsmp-key`. If this file does not exist:

1. Tell the user: "SkillsMP API key is not configured. Get your key at https://skillsmp.com/docs/api#authentication"
2. Ask them to provide the key
3. Save it: `echo "sk_live_..." > ~/.claude/skillsmp-key`
4. Set permissions: `chmod 600 ~/.claude/skillsmp-key`

Read the key on each invocation:

```bash
cat ~/.claude/skillsmp-key 2>/dev/null
```

If the file exists but the API returns 401, the key is invalid — tell the user to update `~/.claude/skillsmp-key`.

## Search

### Step 1: Determine search type

Ask the user which search mode to use (if not obvious from context):

- **Keyword search** (`/api/v1/skills/search`): For exact terms, skill names, categories. Good when the user knows what they're looking for.
- **AI search** (`/api/v1/skills/ai-search`): For natural language descriptions. Good when the user describes what they need without knowing the exact name. Requires API key.

### Step 2: Build and execute the request

**Keyword search:**

```bash
curl -s -X GET "https://skillsmp.com/api/v1/skills/search?q=<URL_ENCODED_QUERY>&limit=20&sortBy=stars" \
  -H "Authorization: Bearer $(cat ~/.claude/skillsmp-key 2>/dev/null)"
```

Additional optional params: `page`, `category`, `occupation`.

**AI search:**

```bash
curl -s -X GET "https://skillsmp.com/api/v1/skills/ai-search?q=<URL_ENCODED_QUERY>" \
  -H "Authorization: Bearer $(cat ~/.claude/skillsmp-key 2>/dev/null)"
```

### Step 3: Parse and present results

Present each result clearly:

```
## Search Results for "<query>"

| # | Name | Author | Stars | Description |
|---|------|--------|-------|-------------|
| 1 | ... | ... | ... | ... |

To install a skill, say "install skill <number>" or "install skill <name>"
```

Show at least these fields when available: name, description, author, stars, category, github_url/repo.

If no results, suggest trying different search terms or switching to AI search.

### Error handling

| Status/Error               | Action                                                   |
| -------------------------- | -------------------------------------------------------- |
| 401 `MISSING_API_KEY`      | Tell user to provide API key                             |
| 401 `INVALID_API_KEY`      | Key is wrong — update `~/.claude/skillsmp-key`           |
| 429 `DAILY_QUOTA_EXCEEDED` | Rate limit hit, try again tomorrow or use anonymous mode |
| 500 `INTERNAL_ERROR`       | Retry once, if still fails report to user                |

## Install

After user selects a skill from search results:

### Step 1: Get the GitHub repo

Search results typically include a GitHub repository URL. If not present, parse the skill page on skillsmp.com to find it:

```bash
curl -s "https://skillsmp.com/skills/<skill-slug>" | grep -oP 'github\.com/[^"<>]+' | head -1
```

### Step 2: Determine install method

Skills on SkillsMP can be:

1. **Standalone GitHub repos** (most common): Repos containing a single skill with SKILL.md
2. **Part of a marketplace**: A collection repo with multiple skills

Check if the repo has a `.claude-plugin/plugin.json` or `marketplace.json`:

```bash
gh api repos/<owner>/<repo>/contents/.claude-plugin/plugin.json --jq '.name' 2>/dev/null
```

If it's a marketplace repo with multiple skills, list them:

```bash
gh api repos/<owner>/<repo>/contents/skills --jq '.[].name' 2>/dev/null
```

### Step 3: Install

**If it's a standalone skill repo:**

```
/plugin marketplace add <owner>/<repo>
/plugin install <skill-name>@<repo-name>
/reload-plugins
```

**If it's part of an existing marketplace the user already has:**

```
/plugin install <skill-name>@<marketplace-name>
/reload-plugins
```

### Step 4: Confirm

After installation, verify by checking the skill appears:

```
/plugin list
```

Tell the user: "Skill <name> installed. Run `/reload-plugins` if it doesn't appear in `/skills`."

## Reference: SkillsMP API

See `[API Documentation - SkillsMP](https://skillsmp.com/docs/api)` for the full API documentation.

Key limits:

- Anonymous: 50 req/day, 10 req/min, keyword search only
- Authenticated: 500 req/day, 30 req/min, all endpoints
- Response headers `X-RateLimit-Daily-Limit` and `X-RateLimit-Daily-Remaining` track quota
