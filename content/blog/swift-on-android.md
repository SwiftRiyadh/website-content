---
slug: "swift-on-android"
title: "Swift on Android: From Dream to Reality with Swift 6.3"
date: "2026-03-30"
author: "Abdullah Alhaider"
github: "cs4alhaider"
twitter: "cs4alhaider"
tags: ["swift", "android", "cross-platform", "skip", "swift 6.3"]
excerpt: "Swift 6.3 officially ships with Android as a supported platform. Here's what that means, how it works, and how Skip.dev takes it even further with full SwiftUI-to-Compose bridging."
---

## Swift Now Runs on Android — Officially

On March 26, 2026, [Swift 6.3 was released](https://www.swift.org/blog/swift-6.3-released/) with a milestone that many developers thought would never come: **Android is now a first-class supported platform**, alongside macOS, iOS, Windows, and Linux.

The Swift SDK for Android ships as an officially maintained target — just like the Static Linux and WebAssembly SDKs. This means you can compile Swift code directly to native Android binaries, with performance comparable to C/C++ code built with the Android NDK.

This didn't happen overnight. A community effort spanning over a decade — led by companies like [Readdle](https://readdle.com) (Spark email) and [flowkey](https://www.flowkey.com) — laid the groundwork. In February 2025, the **Swift on Android Community Working Group** was founded, and it later gained official workgroup status from the Swift Platform Steering Group.

## How Does It Work?

Swift on Android compiles your code to **native machine code** — there's no interpreter, no VM, and no transpilation layer at this level. Your app bundles a native Swift runtime that includes the standard library, Dispatch, and Foundation.

Since Android APIs are primarily exposed through Java and Kotlin, Swift uses the **Java Native Interface (JNI)** for interoperability. The Swift project provides two key tools:

- **`jextract`** — automatically generates Swift bindings from Java APIs
- **`wrap-java`** — creates wrappers that let you call Swift from Java or Java from Swift

Here's a simple example of using Android API versioning in Swift — something that landed with the SDK:

```swift
@available(Android 33, *)
func backtrace() {
    withUnsafeTemporaryAllocation(
        of: UnsafeMutableRawPointer.self,
        capacity: 1
    ) { address in
        _ = backtrace(address.baseAddress!, 1)
    }
}
```

You can also use `#available` runtime checks, just like you would on iOS:

```swift
if #available(Android 33, *) {
    // Use API available on Android 13+
} else {
    // Fallback for older versions
}
```

## Getting Started with the Swift SDK for Android

The official [Getting Started guide](https://www.swift.org/documentation/articles/swift-sdk-for-android-getting-started.html) walks you through the setup. At a high level:

1. Install the Swift 6.3 toolchain
2. Install the Android SDK for Swift
3. Cross-compile your Swift package for Android targets
4. Use GitHub Actions workflows to build and test in Android emulators

A basic Swift package that builds for Android looks like any other Swift package:

```swift
// Package.swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "MyAndroidLib",
    products: [
        .library(name: "MyAndroidLib", targets: ["MyAndroidLib"])
    ],
    targets: [
        .target(name: "MyAndroidLib"),
        .testTarget(name: "MyAndroidLibTests", dependencies: ["MyAndroidLib"])
    ]
)
```

Then build for Android:

```bash
swift build --swift-sdk android-arm64
```

Over **2,200 Swift packages** already build for Android, including popular ones like Alamofire, SwiftSoup, swift-protobuf, and flatbuffers.

## Apps Already in Production

Several well-known apps have been running Swift on Android in production for years, proving the approach works at scale:

- **Spark** by Readdle — email client
- **flowkey** — piano learning app
- **MediQuo** — healthcare platform
- **Naturitas** — e-commerce marketplace

## Enter Skip: SwiftUI on Android

While the official SDK handles Swift code compilation, it deliberately does not provide a GUI solution. That's where [Skip](https://skip.dev) comes in.

**Skip** is an open-source framework that lets you write one app in Swift and SwiftUI and ship it natively on both iOS and Android. It was co-founded by the same people who helped establish the Swift on Android Working Group.

The key difference from other cross-platform solutions: Skip produces **genuinely native apps** on both platforms. There's no embedded rendering engine, no WebView, and no Flutter-style widget layer.

- On **iOS**: your code runs as pure SwiftUI — zero Skip dependency at runtime
- On **Android**: Swift compiles to native code, and SwiftUI declarations are bridged to **Jetpack Compose**

### Skip's Two Modes

**Lite Mode** — transpiles Swift to Kotlin and SwiftUI to Compose. Good for simpler apps and UI-heavy code.

**Fuse Mode** — uses the official Swift 6.3 SDK to compile Swift natively on Android, with SwiftUI bridged to Compose through **SkipFuseUI**. This is the recommended mode as of Skip 1.8, and it delivers memory usage and battery consumption comparable to hand-written Kotlin.

### A Taste of Skip in Action

Here's a simple SwiftUI view that runs on both iOS (SwiftUI) and Android (Compose) through Skip:

```swift
import SwiftUI

struct WelcomeView: View {
    @State private var name = ""

    var body: some View {
        VStack(spacing: 20) {
            Text("Welcome to Swift Riyadh!")
                .font(.largeTitle)
                .fontWeight(.bold)

            TextField("Enter your name", text: $name)
                .textFieldStyle(.roundedBorder)
                .padding(.horizontal)

            if !name.isEmpty {
                Text("Hello, \(name)! 👋")
                    .font(.title2)
            }
        }
        .padding()
    }
}
```

This renders as native SwiftUI on iOS and native Jetpack Compose on Android — no changes needed.

### Interop with Kotlin and Compose

Skip provides powerful interoperability tools when you need platform-specific behavior:

```swift
import SkipFuse

// Call Kotlin/Java APIs from Swift
let context = AnyDynamicObject(className: "android.content.Context")

// Embed Jetpack Compose directly in SwiftUI
struct MyView: View {
    var body: some View {
        #if SKIP
        ComposeView { context in
            // Write Kotlin/Compose code here
        }
        #else
        Text("iOS-specific view")
        #endif
    }
}
```

You can also write inline Kotlin inside `#if SKIP` blocks, making it easy to access Android-specific APIs when needed.

## What This Means for Swift Developers

The combination of Swift 6.3's official Android SDK and Skip's SwiftUI bridging gives Swift developers two powerful options:

1. **Share business logic** — cross-compile your Swift model layer, networking, and data logic for Android, while writing native UI per platform
2. **Share everything** — write your entire app in Swift and SwiftUI, and let Skip generate the Android version

Either way, you're writing Swift. Your type safety, your concurrency model, your existing packages — they all come with you.

It's worth noting: **Kotlin isn't going anywhere**. It remains the primary language for Android development. But for teams already invested in the Apple ecosystem, Swift on Android removes the need to maintain two completely separate codebases.

## Current Limitations

The ecosystem is maturing rapidly, but there are a few things to be aware of:

- **Binary size**: the Swift runtime on Android is currently around 60 MB, though this is expected to shrink
- **Debugging**: tooling for debugging Swift on Android is still being improved, with integration planned for VS Code and Android Studio
- **UI framework**: the official SDK doesn't include a UI framework — you'll need Skip or another solution for cross-platform UI

## Resources

- [Exploring the Swift SDK for Android](https://www.swift.org/blog/exploring-the-swift-sdk-for-android/) — official Swift blog post
- [Getting Started Guide](https://www.swift.org/documentation/articles/swift-sdk-for-android-getting-started.html) — setup instructions
- [Swift 6.3 Release Notes](https://www.swift.org/blog/swift-6.3-released/) — full release details
- [Skip Documentation](https://skip.dev/docs/) — getting started with Skip
- [Skip on GitHub](https://github.com/skiptools/skip) — open-source repository
- [Swift 6.3 Android Support](https://skip.dev/blog/swift-63-android-support/) — Skip's perspective on the release

## Wrapping Up

Swift on Android has gone from a community experiment to an officially supported platform in Swift 6.3. Whether you're sharing business logic across platforms or building full apps with SwiftUI through Skip, the tooling is here and production-ready.

The Swift ecosystem just got a lot bigger. And for Swift developers in Riyadh and beyond, the question is no longer "can I use Swift on Android?" — it's "how do I want to use it?"

*— Abdullah Alhaider*
