---
slug: "apple-foundation-models-on-device-ai"
title: "Apple's Foundation Models Framework: On-Device AI for Swift Developers"
date: "2026-03-22"
author: "Abdullah Alhaider"
github: "cs4alhaider"
twitter: "cs4alhaider"
tags: ["apple intelligence", "foundation models", "machine learning", "swift"]
excerpt: "Apple introduced the Foundation Models framework, giving developers direct access to the on-device large language model powering Apple Intelligence. Here's what you need to know."
---

## A New Era for On-Device Intelligence

At WWDC25, Apple unveiled the **Foundation Models** framework — a new way for developers to tap into the same on-device large language model that powers Apple Intelligence. Available starting with iOS 26, macOS 26, iPadOS 26, and visionOS 26, this framework lets you build intelligent features that run entirely on-device, with no server round-trips and full user privacy.

For Swift developers, this is a game-changer. Instead of integrating third-party AI APIs and managing cloud infrastructure, you can now leverage Apple's built-in model directly from your Swift code.

## What Can It Do?

The on-device model specializes in **language understanding and generation**. Here are some of the tasks it excels at:

- **Summarization** — Condense long text into concise summaries
- **Entity extraction** — Pull structured data from unstructured text
- **Text refinement** — Improve grammar, tone, or style of written content
- **Creative content** — Generate dialog for games, stories, or conversational UI
- **Content categorization** — Tag and organize data intelligently

## How It Works

The API is refreshingly simple. You create a `LanguageModelSession`, give it a prompt, and get a response:

```swift
import FoundationModels

let session = LanguageModelSession()
let response = try await session.respond(to: "Summarize the key features of SwiftUI in three sentences.")
print(response)
```

You can also provide `Instructions` to steer the model's behavior across an entire session — defining its role, style, and safety guardrails.

## Guided Generation with @Generable

One of the most powerful features is **guided generation**. Instead of parsing raw text output, you can define a Swift struct with the `@Generable` macro and the framework guarantees the model returns an instance of your type:

```swift
import FoundationModels

@Generable
struct RestaurantReview {
    var sentiment: String
    var score: Int
    var summary: String
}
```

This eliminates fragile string parsing and gives you type-safe AI output — a very Swift approach to the problem.

## Tool Calling

The framework supports **tool calling**, where the model can invoke code you write to extend its capabilities. Define a `Tool` that searches a database, calls an API, or performs a calculation, and the model will decide when to use it:

This opens up possibilities like letting the model query your app's local data, fetch calendar events, or trigger actions — all while keeping the intelligence loop on-device.

## Context Window and Limitations

The on-device model supports up to **4,096 tokens** per session. A token is roughly 3-4 characters in English. The total of your instructions, prompts, and outputs must fit within this window.

For larger tasks, Apple recommends breaking data into chunks, processing each in a separate session, and combining results. If you exceed the limit, the framework throws `exceededContextWindowSize` — so plan your prompts accordingly.

## Requirements

- Device must support **Apple Intelligence**
- Apple Intelligence must be **enabled** in Settings
- Available on **iOS 26+, macOS 26+, iPadOS 26+, visionOS 26+**

Always check model availability with `SystemLanguageModel.default` and provide a fallback experience for unsupported devices.

## What This Means for the Community

The Foundation Models framework lowers the barrier for adding AI features to Swift apps dramatically. No API keys, no cloud costs, no latency — just native Swift code running a powerful language model on the user's device.

At Swift Riyadh, we're excited to explore this in upcoming workshops. If you're building apps that could benefit from summarization, smart suggestions, content generation, or structured data extraction, this framework deserves your attention.

Stay tuned for a hands-on workshop where we'll build a real feature using Foundation Models together.

*— The Swift Riyadh Team*
