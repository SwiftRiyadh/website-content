# Swift Riyadh — Website Content

This repo contains only markdown content (blog posts and events) for [SwiftRiyadh.com](https://SwiftRiyadh.com). Content is synced to the website repo via GitHub Actions on merge to `main`.

## PR Review Instructions

When reviewing a PR, validate every added or modified `.md` file under `content/` against the rules below. If any check fails, request changes with a clear explanation.

## Content Validation Rules

### Blog Post (`content/blog/*.md`)

**Required frontmatter fields:**
- `slug` — URL-safe (lowercase letters, numbers, and hyphens only), must match the filename without `.md`
- `title` — non-empty string
- `date` — must be in `YYYY-MM-DD` format and wrapped in quotes
- `author` — non-empty string
- `tags` — JSON array with at least one tag, e.g. `["swift", "tutorial"]`

**Optional frontmatter fields:**
- `excerpt` — brief summary (auto-generated from body if omitted)
- `github` — GitHub username without `@`
- `twitter` — Twitter/X username without `@`
- `cover` — image URL
- `readingTime` — e.g. `"5 min read"` (auto-calculated if omitted)

**Body:**
- Must not be empty (content after the closing `---`)
- Should contain at least one `##` heading

### Event (`content/events/*.md`)

**Required frontmatter fields:**
- `slug` — URL-safe (lowercase letters, numbers, and hyphens only), must match the filename without `.md`
- `title` — non-empty string
- `date` — must be in `YYYY-MM-DD` format and wrapped in quotes
- `time` — time range string, e.g. `"18:30 - 21:00"`
- `location` — venue name and city
- `speakers` — YAML array with at least one entry, each must have a `name`
- `tags` — JSON array with at least one tag
- `rsvpLink` — registration URL (use `"#"` if not available yet)

**Optional frontmatter fields:**
- `locationUrl` — direct maps/directions link
- `cover` — image URL
- Speaker social links: `twitter`, `github`, `linkedin` (without `@`)

**Body:**
- Must not be empty
- Should contain at least one `##` heading

## Validation Checklist

For every content file in the PR, verify:

1. Frontmatter is wrapped in `---` delimiters
2. All required fields are present for the content type (blog vs event)
3. `slug` matches the filename (without `.md`)
4. `slug` is URL-safe: lowercase, hyphens, no spaces or special characters
5. `date` is valid `YYYY-MM-DD` and wrapped in quotes
6. `tags` is a valid JSON array (not a YAML list)
7. Social usernames do not start with `@`
8. Body content exists and is meaningful
9. No duplicate slugs across existing content files in the repo
10. Files are placed in the correct directory (`content/blog/` or `content/events/`)
