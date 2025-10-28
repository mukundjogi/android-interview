---
layout: default
title: "Android KMM - Interview Preparation"
---

# Android KMM (Kotlin Multiplatform Mobile)

## Table of Contents
- [Overview](#overview)
- [Code Sharing](#code-sharing)
- [Expect/Actual](#expectactual)
- [Interview Questions & Answers](#interview-questions--answers)

## Code Sharing

Shared module structure:

```kotlin
// commonMain
class Greeting {
    fun greet(): String = "Hello from KMM"
}

// androidMain
actual class Greeting {
    actual fun greet() = "Hello from Android"
}

// iosMain
actual class Greeting {
    actual fun greet() = "Hello from iOS"
}
```

## Interview Questions & Answers

### Q1: What is Android KMM?

**Answer:**

Kotlin Multiplatform Mobile (KMM) allows sharing business logic between Android and iOS.

**Architecture:**
- Shared module for business logic
- Platform-specific modules for UI
- Expect/actual mechanism for platform APIs

```kotlin
// Shared business logic
expect class Platform() {
    val platform: String
}

// Android implementation
actual class Platform actual constructor() {
    actual val platform: String = "Android"
}
```

### Q2: How does code sharing work in KMM?

**Answer:**

**Conceptual Foundation:**

KMM uses a multi-source set approach where code is organized into platform-specific implementations of common interfaces. The common code defines the expected behavior, while each platform provides the actual implementation.

**Architecture Layers:**

The code is structured into source sets:
- `commonMain`: Contains shared business logic, data models, and platform-agnostic code
- `androidMain`: Android-specific implementations
- `iosMain`: iOS-specific implementations

This separation allows maximum code sharing while maintaining platform flexibility.

**Code Sharing Layers:**

KMM uses a layered approach for code sharing:
- **Common Code**: Shared Kotlin code used by all platforms
- **Platform-Specific Code**: Platform implementations using expect/actual

```kotlin
// commonMain/shared code
expect class Platform() {
    val platform: String
}

class Greeting {
    fun greet(): String {
        return "Hello, ${Platform().platform}!"
    }
}

// androidMain
actual class Platform actual constructor() {
    actual val platform: String = "Android ${android.os.Build.VERSION.SDK_INT}"
}

// iosMain
actual class Platform actual constructor() {
    actual val platform: String = UIDevice.currentDevice.systemName()
}
```

### Q3: Explain expect/actual mechanism in KMM

**Answer:**

**Conceptual Understanding:**

The expect/actual mechanism in KMM provides a powerful way to define platform-specific APIs while maintaining a common interface. This is Kotlin's way of implementing the "platform-specific API" pattern.

**How It Works:**

1. **Expect Declaration**: In `commonMain`, you declare an expected class or function with the `expect` keyword
2. **Actual Implementation**: In each platform's source set (`androidMain`, `iosMain`), you provide the actual implementation
3. **Compile-time Safety**: The compiler ensures that every `expect` has a matching `actual` in each platform

**Benefits:**
- Type-safe platform-specific code
- Compiler-enforced completeness
- Clean separation of concerns
- Maximum code sharing

**Expect/Actual Pattern:**

The expect/actual mechanism allows you to define platform-specific APIs.

```kotlin
// commonMain
expect class FileSystem() {
    fun readFile(path: String): String
}

// androidMain
actual class FileSystem actual constructor() {
    actual fun readFile(path: String): String {
        val context = AppContext.get()
        return context.assets.open(path).bufferedReader().use { it.readText() }
    }
}

// iosMain
actual class FileSystem actual constructor() {
    actual fun readFile(path: String): String {
        return NSString.create(NSBundle.mainBundle().pathForResource(path, "txt")!!).string()
    }
}
```

### Q4: What are the limitations of KMM?

**Answer:**

**Conceptual Understanding:**

While KMM is powerful for code sharing, it has inherent limitations due to platform differences and the nature of mobile development. Understanding these limitations helps make informed decisions about its use.

**Fundamental Constraints:**
- Cannot share UI code (platforms have different UI systems)
- UI must be platform-native
- Platform-specific features require separate implementations
- Some Kotlin features not available on all platforms

**Practical Limitations:**
- Learning curve for both Android and iOS teams
- Debugging iOS code requires Xcode
- Smaller community compared to native frameworks
- Platform parity not 100% guaranteed

**Key Limitations:**
- UI is platform-specific (can't share UI code)
- Learning curve for iOS developers
- Smaller community than native frameworks
- Debugging on iOS requires Xcode
- Some Kotlin features not supported on iOS

### Q5: Compare KMM with Flutter

**Answer:**

**Conceptual Comparison:**

Both KMM and Flutter are approaches to multi-platform development, but they follow different philosophies:

**KMM:** Share business logic, implement native UI
**Flutter:** Share everything including UI using a custom rendering engine

**Key Differences:**

- **Code Sharing:** KMM shares logic only, Flutter shares UI + logic
- **Approach:** KMM = minimal sharing, Flutter = maximal sharing
- **Performance:** KMM = native performance, Flutter = custom rendering overhead
- **Learning Curve:** KMM = learn platform UIs, Flutter = learn Dart + Flutter
- **Use Cases:** KMM = complex apps needing native feel, Flutter = rapid development across platforms

| Aspect | KMM | Flutter |
|--------|-----|---------|
| UI Sharing | No | Yes |
| Language | Kotlin | Dart |
| Performance | Native | Custom rendering |
| Code Sharing | Logic only | UI + Logic |
| Learning Curve | Medium | Steep |

### Q6: How to handle platform-specific code in KMM?

**Answer:**

```kotlin
// commonMain
expect class NetworkProvider() {
    fun getRequest(url: String): String
}

// androidMain
actual class NetworkProvider actual constructor() {
    actual fun getRequest(url: String): String {
        val urlConnection = URL(url).openConnection()
        return urlConnection.inputStream.bufferedReader().use { it.readText() }
    }
}

// iosMain
actual class NetworkProvider actual constructor() {
    actual fun getRequest(url: String): String {
        return NSString.stringWithContentsOfURL(NSURL.URLWithString(url))
    }
}
```

### Q7: What is the use case for KMM?

**Answer:**

**Conceptual Framework:**

KMM is best suited for projects where you want to share business logic while maintaining native UI experiences. It's not a one-size-fits-all solution.

**Ideal Scenarios:**
- Existing Android app adding iOS support
- Teams familiar with Kotlin
- Apps requiring platform-specific UI optimizations
- Complex business logic that benefits from sharing

**When to Consider KMM:**
- You have significant business logic to share
- Team includes Kotlin developers
- Need for platform-specific optimizations
- Want to leverage native platform features

**Best Use Cases:**
- Business logic sharing between Android and iOS
- Data layer sharing
- Networking layer
- Database layer

**Not Suitable For:**
- UI sharing
- Simple single-platform apps
- Prototyping/MVPs

### Q8: How to structure a KMM project?

**Answer:**

Design the project to clearly separate shared business logic from platform-specific UI.
Keep dependencies flowing from platform modules into the shared module to avoid leaks.

**Conceptual Understanding:**

KMM project structure separates shared and platform-specific code while maintaining clear module boundaries. This structure enables code sharing while keeping platform implementations separate.

**Directory Organization:**
- `commonMain`: Shared business logic, data models
- `commonTest`: Shared unit tests
- `androidMain`: Android-specific implementations
- `iosMain`: iOS-specific implementations

**Module Responsibilities:**
- Shared modules contain business logic
- Platform modules contain UI and platform APIs
- Dependencies flow from platform to shared

**Project Structure:**
```
shared/
  commonMain/
    kotlin/
      business logic
  androidMain/
    kotlin/
      Android-specific
  iosMain/
    kotlin/
      iOS-specific

androidApp/
  MainActivity.kt

iosApp/
  AppDelegate.swift
```

---

**Previous:** [MVI Pattern](mvi-pattern.md)

**Next:** [Coroutines & Flows](coroutines-flows.md)

