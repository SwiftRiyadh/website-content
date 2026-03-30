---
slug: "swift-package-collections"
title: "Swift Package Collections: Curate and Share Your Dependencies"
date: "2026-03-30"
author: "Abdullah Alhaider"
github: "cs4alhaider"
twitter: "cs4alhaider"
tags: ["swift", "spm", "packages", "tutorial"]
excerpt: "Learn how to create, publish, and use Swift Package Collections — a powerful way to curate and discover Swift packages for your team or community."
---

## What Are Swift Package Collections?

Swift Package Collections, introduced in [SE-0291](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0291-package-collections.md), let you group Swift packages into curated lists that can be shared and imported into Xcode or the Swift Package Manager CLI. Think of them as playlists for Swift packages.

Instead of searching the internet for packages or maintaining a shared document of recommended dependencies, you can create a `.json` collection that anyone can subscribe to.

This is especially useful for:

- **Teams** — Standardize approved dependencies across your org
- **Communities** — Share curated lists of packages for specific domains (e.g., networking, UI, server-side Swift)
- **Educators** — Provide students with a ready-made set of packages for a course

## How It Works

A package collection is a signed JSON file hosted at a URL. It contains metadata about each package: name, description, versions, and repository URL. Users add the collection URL in Xcode or the `swift package-collection` CLI, and all listed packages become discoverable.

```
┌─────────────────────┐
│  Collection Author  │
│  (you)              │
├─────────────────────┤
│  1. Define packages │
│  2. Generate JSON   │
│  3. Sign & host     │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Collection URL     │
│  (hosted JSON file) │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Consumer           │
│  (Xcode / CLI)      │
├─────────────────────┤
│  Add collection URL │
│  Browse & add pkgs  │
└─────────────────────┘
```

## Creating a Package Collection

### Step 1: Install the Generator

Apple provides `package-collection-generate` as part of the [swift-package-collection-generator](https://github.com/apple/swift-package-collection-generator) repo:

```bash
git clone https://github.com/apple/swift-package-collection-generator.git
cd swift-package-collection-generator
swift build -c release
```

The binary is at `.build/release/package-collection-generate`.

### Step 2: Define Your Input

Create a `collection-input.json` file listing the packages you want to include:

```json
{
  "name": "Swift Riyadh Favorites",
  "overview": "A curated list of Swift packages recommended by the Swift Riyadh community.",
  "keywords": ["swift", "ios", "server-side", "community"],
  "packages": [
    {
      "url": "https://github.com/Alamofire/Alamofire.git",
      "summary": "Elegant HTTP networking in Swift."
    },
    {
      "url": "https://github.com/pointfreeco/swift-composable-architecture.git",
      "summary": "A library for building applications in a consistent and understandable way."
    },
    {
      "url": "https://github.com/vapor/vapor.git",
      "summary": "A server-side Swift HTTP web framework."
    },
    {
      "url": "https://github.com/onevcat/Kingfisher.git",
      "summary": "A lightweight, pure-Swift library for downloading and caching images."
    },
    {
      "url": "https://github.com/SwiftyJSON/SwiftyJSON.git",
      "summary": "The better way to deal with JSON data in Swift."
    }
  ]
}
```

### Step 3: Generate the Collection

Run the generator, pointing it at your input file:

```bash
.build/release/package-collection-generate collection-input.json collection-output.json
```

The generator fetches metadata (versions, manifests, README) from each package's Git repository and produces a complete `collection-output.json`.

### Step 4: Sign the Collection (Recommended)

Signing establishes trust. Xcode shows a warning for unsigned collections. You need an Apple Developer certificate (the same one you use for code signing):

```bash
.build/release/package-collection-sign \
  collection-output.json \
  collection-signed.json \
  --certificate-path /path/to/certificate.cer \
  --key-path /path/to/private-key.pem
```

If you don't have a certificate handy, you can skip signing for local testing — but always sign for public distribution.

### Step 5: Host the Collection

Upload the signed JSON file to any HTTPS-accessible URL:

- **GitHub Pages** — Commit to a repo with Pages enabled
- **Cloudflare R2 / S3** — Upload to a bucket with public access
- **Your own server** — Any static file hosting works

Example URL: `https://swiftriyadh.com/packages/collection.json`

## Using a Package Collection

### In Xcode

1. Open Xcode → **File → Add Package Dependencies...**
2. Click the **+** at the bottom-left of the package browser
3. Select **Add Package Collection...**
4. Paste the collection URL
5. Browse and add packages from the collection

### With the CLI

```bash
# Add a collection
swift package-collection add https://swiftriyadh.com/packages/collection.json

# List all collections
swift package-collection list

# Search within collections
swift package-collection search --keywords networking

# Get info about a specific package
swift package-collection describe https://github.com/Alamofire/Alamofire.git

# Remove a collection
swift package-collection remove https://swiftriyadh.com/packages/collection.json
```

## Real-World Use Cases

### Team Standards

Create a collection of approved packages for your organization. New team members add one URL and instantly see every dependency your team trusts:

```json
{
  "name": "Acme Corp Approved Packages",
  "overview": "Packages vetted by the iOS platform team. Use these first before adding new dependencies.",
  "packages": [
    { "url": "https://github.com/Alamofire/Alamofire.git" },
    { "url": "https://github.com/realm/realm-swift.git" },
    { "url": "https://github.com/firebase/firebase-ios-sdk.git" }
  ]
}
```

### Community Curation

The Swift Riyadh community could maintain a collection of packages used in meetup demos, workshops, and open-source projects. Anyone in the community can submit a PR to add a package.

### Domain-Specific Lists

Create focused collections for specific use cases:

- **Server-Side Swift** — Vapor, Hummingbird, SwiftNIO, Fluent, Redis
- **SwiftUI Toolkit** — Navigation, state management, UI components
- **Testing & CI** — SnapshotTesting, Quick, Nimble, OHHTTPStubs

## Automating Collection Updates

You can automate regeneration with a GitHub Action that runs whenever the input file changes:

```yaml
name: Update Package Collection

on:
  push:
    branches: [main]
    paths: ['collection-input.json']

jobs:
  generate:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Generator
        run: |
          git clone https://github.com/apple/swift-package-collection-generator.git /tmp/gen
          cd /tmp/gen && swift build -c release

      - name: Generate Collection
        run: |
          /tmp/gen/.build/release/package-collection-generate \
            collection-input.json docs/collection.json

      - name: Commit and Push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add docs/collection.json
          git diff --staged --quiet || git commit -m "update: regenerate package collection"
          git push
```

Host the `docs/` folder via GitHub Pages and your collection stays up to date automatically.

## Tips

- **Keep collections focused** — A collection of 10-20 packages is more useful than one with hundreds
- **Add summaries** — The `summary` field shows up in Xcode's package browser, making it easier for developers to pick the right package
- **Version your collections** — Use semantic versioning in the collection metadata so consumers know when packages were added or removed
- **Sign for trust** — Unsigned collections trigger warnings in Xcode and may be rejected by organizations with strict security policies

## Wrapping Up

Swift Package Collections bridge the gap between "I know this package exists" and "my whole team uses it." Whether you're standardizing dependencies for a company, curating packages for a community, or building a teaching toolkit — collections make package discovery structured and shareable.

Try creating one for your team this week. Start small, host it on GitHub Pages, and share the URL in your team's chat.

*— Abdullah Alhaider*
