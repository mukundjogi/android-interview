---
layout: default
title: "Gradle & Maven - Interview Preparation"
---

# Gradle & Maven

## Table of Contents
- [Gradle Basics](#gradle-basics)
- [Build Variants](#build-variants)
- [ProGuard vs R8](#proguard-vs-r8)
- [Interview Questions & Answers](#interview-questions--answers)

## Build Variants

```gradle
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
        debug {
            applicationIdSuffix ".debug"
            debuggable true
        }
    }
    
    flavorDimensions "environment"
    productFlavors {
        dev {
            dimension "environment"
            applicationIdSuffix ".dev"
        }
        prod {
            dimension "environment"
        }
    }
}
```

## Interview Questions & Answers

### Q1: Compare Gradle and Maven

**Answer:**

**Gradle:**
- Build automation tool
- Uses Groovy or Kotlin DSL
- Supports incremental builds
- Better Android integration

**Maven:**
- Project management tool
- XML-based configuration
- More verbose
- Not commonly used for Android

### Q2: What is the difference between implementation and api?

**Answer:**

**Conceptual Understanding:**

The difference between `implementation` and `api` is crucial for Gradle's dependency management and affects build performance and encapsulation.

**Key Concepts:**

`implementation` (Recommended):
- Dependency is internal to the module
- Not exposed to modules that depend on this module
- Faster incremental builds (only recompiles this module on change)
- Better encapsulation

`api`:
- Dependency is exposed to consumers
- Changes in transitive dependencies trigger recompilation of all dependent modules
- Use when other modules need to access this dependency's types

**When to Use Each:**
- Use `implementation` for dependencies used only internally
- Use `api` only when other modules need access to dependency's public API

```gradle
// implementation - dependency is private to the module
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
}

// api - dependency is exposed to consumers
dependencies {
    api 'com.squareup.retrofit2:retrofit:2.9.0'
}
```

**Key Difference:**
- `implementation`: Dependency is not leaked to consumers
- `api`: Dependency is exposed to consumers

### Q3: How to configure build variants and flavors?

**Answer:**

```gradle
android {
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
            debuggable true
        }
        release {
            minifyEnabled true
            signingConfig signingConfigs.release
        }
    }
    
    flavorDimensions "version", "api"
    productFlavors {
        demo {
            dimension "version"
            applicationId "com.example.demo"
        }
        full {
            dimension "version"
            applicationId "com.example.full"
        }
        
        minApi21 {
            dimension "api"
            minSdkVersion 21
        }
        minApi24 {
            dimension "api"
            minSdkVersion 24
        }
    }
}
```

### Q4: Compare ProGuard and R8

**Answer:**

**Conceptual Understanding:**

Both ProGuard and R8 are code shrinking and obfuscation tools that reduce app size and make reverse engineering harder by removing unused code and renaming classes/methods.

**The Need for Code Shrinking:**
- Reduce app size by removing unused code
- Obfuscate code to prevent reverse engineering
- Optimize bytecode for better performance
- Remove logging and debug code from releases

**Evolution:**
- ProGuard was the original tool, separate from Android build system
- R8 was developed by Google specifically for Android, integrating ProGuard's functionality
- R8 is now the default for Android apps

**R8 (Recommended):**
- Faster than ProGuard
- Better code shrinking
- Integrated in Android Gradle
- Single tool for shrinking/obfuscation

**ProGuard:**
- Older tool
- Separate from Android
- Less optimized

### Q5: How to optimize Gradle build performance?

**Answer:**

**Build Performance Philosophy:**

Gradle build performance directly impacts developer productivity. Slow builds mean longer feedback loops and wasted developer time.

**Performance Bottlenecks:**
- Dependency resolution
- Compilation of unchanged modules
- Task execution
- JVM startup time
- Disk I/O operations

**Optimization Strategies:**

**1. Enable build cache:**
```gradle
gradle.properties:
org.gradle.caching=true
```

**2. Enable parallel builds:**
```gradle
org.gradle.parallel=true
```

**3. Use implementation instead of api:**
```gradle
dependencies {
    implementation '...' // Faster than api
}
```

**4. Enable daemon:**
```gradle
org.gradle.daemon=true
```

### Q6: How to create custom Gradle tasks?

**Answer:**

**Conceptual Understanding:**

Gradle tasks are atomic units of work that can be executed during the build process. Custom tasks allow you to extend the build system to perform specific operations like copying files, generating code, or running custom scripts.

**Task Lifecycle:**
1. Configuration Phase: Tasks are created and configured
2. Execution Phase: Tasks are executed in dependency order
3. Up-to-date Check: Skipped if nothing changed

**Common Use Cases:**
- Code generation
- File manipulation
- API documentation
- Custom build steps
- Deployment automation

```gradle
tasks.register("myTask") {
    doLast {
        println("My custom task")
    }
}

// Task with dependencies
tasks.register("compileAndTest") {
    dependsOn "compile", "test"
    doLast {
        println("Compile and test completed")
    }
}

// Task in Kotlin DSL
tasks {
    register("copyFiles") {
        doLast {
            copy {
                from("src")
                into("dest")
            }
        }
    }
}
```

### Q7: Explain multi-module Gradle project structure

**Answer:**

**Conceptual Foundation:**

Multi-module Gradle projects organize code into logical, independently buildable modules. Each module can have its own dependencies, build configuration, and lifecycle.

**Module Dependencies:**
- Dependencies flow in one direction
- Core modules provide base functionality
- Feature modules depend on core modules
- App module depends on all modules

**Benefits:**
- Encapsulation of functionality
- Parallel builds
- Clear module boundaries
- Reusable components

```
:app
  └── implementation project(':feature:home')
  └── implementation project(':core:network')

:feature:home
  └── implementation project(':core:ui')

:core:network
  └── implementation project(':core:database')

:core:database
```

**settings.gradle:**
```gradle
include ':app', ':feature:home', ':core:network', ':core:database'
```

### Q8: What are Gradle configurations?

**Answer:**

Gradle configurations define how dependencies are used at compile time and runtime.
Choose the narrowest scope (e.g., implementation vs api) to optimize build times and encapsulation.

**Common Configurations:**
- `implementation`: Compile + runtime
- `api`: Exposed to consumers
- `compileOnly`: Compile time only
- `runtimeOnly`: Runtime only
- `testImplementation`: Test dependencies
- `androidTestImplementation`: Android test dependencies

---

**Previous:** [Testing](testing.md)

**Next:** [Publishing to Play Store](publishing-play-store.md)

