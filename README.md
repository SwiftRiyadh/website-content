# 📝 Website Content

Content source for [SwiftRiyadh.com](https://SwiftRiyadh.com). Blogs and events in markdown, auto-deployed on merge.

---

## How It Works

```mermaid
graph LR
    A["✍️ Author"] -->|"Open PR"| B["swiftriyadh/\nwebsite-content"]
    B -->|"PR merged"| C["GitHub Action\n(dispatch)"]
    C -->|"repository_dispatch"| D["swiftriyadh/\nwebsite"]
    D -->|"Auto-detected"| E["Cloudflare Pages\n(build + deploy)"]
    E --> F["🌐 SwiftRiyadh.com"]
```

1. Open a PR with your new blog or event markdown
2. PR gets reviewed and merged to `main`
3. GitHub Actions sends a `repository_dispatch` to the website repo
4. Website repo pulls the latest content and commits it
5. Cloudflare Pages auto-detects the commit and deploys

---

## Tools

| Tool | Description |
|------|-------------|
| [Content Creator](https://swiftriyadh.com/tools/content-creator) | Create blog posts and events with frontmatter, preview, and export. Supports creating from scratch, importing existing `.md` files, or pasting raw markdown. |
| [AI Skill Generator](https://swiftriyadh.com/tools/ai-skill) | Copy a ready-made prompt for your AI assistant so it can write content in the correct format — available for both blog posts and events. |
| [Markdown Validator](https://swiftriyadh.com/tools/validator) | Paste your markdown and validate that all required frontmatter fields are present before submitting a PR. |

---

## Contributing

1. Create a new branch
2. Add your markdown file in `content/blogs/` or `content/events/`
3. Use the [Content Creator](https://swiftriyadh.com/tools/content-creator) or write markdown manually following the formats above
4. Validate your content with the [Markdown Validator](https://swiftriyadh.com/tools/validator)
5. Open a PR — once merged, it goes live automatically
