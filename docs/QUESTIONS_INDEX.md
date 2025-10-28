---
layout: default
title: "Questions Index - Android Interview Preparation"
---

# Questions Index

Complete index of all interview questions across topics.

## Topics

- [Introduction](#introduction-7-questions)
- [Android Basics](#android-basics-7-questions)
- [Kotlin Programming](#kotlin-programming-8-questions)
- [Java vs Kotlin](#java-vs-kotlin-10-questions)
- [Android Architecture](#android-architecture-12-questions)
- [Jetpack Compose](#jetpack-compose-10-questions)
- [MVI Pattern](#mvi-pattern-8-questions)
- [Android KMM](#android-kmm-8-questions)
- [Coroutines & Flows](#coroutines--flows-10-questions)
- [Data Persistence](#data-persistence-8-questions)
- [Networking](#networking-8-questions)
- [Dependency Injection](#dependency-injection-8-questions)
- [Memory Management](#memory-management-8-questions)
- [Testing](#testing-7-questions)
- [Gradle & Maven](#gradle--maven-8-questions)
- [Publishing to Play Store](#publishing-to-play-store-9-questions)
- [Security](#security-8-questions)
- [Advanced Topics](#advanced-topics-8-questions)

## Introduction (7 Questions)

1. How should I approach an Android interview?
2. What topics are most frequently asked in Android interviews?
3. How long does interview preparation take?
4. What should I focus on: theory or practice?
5. How do I handle questions I don't know?
6. Should I only prepare for Android-specific topics?
7. What projects should I build for my portfolio?

## Android Basics (7 Questions)

1. Explain the Activity Lifecycle and when each method is called
2. What is the difference between Activity and Fragment lifecycles?
3. How do you handle configuration changes in Android?
4. What is an Intent and what are the types of Intents?
5. Explain Android permissions system
6. What are the different types of Context and when to use them?
7. What is the difference between onResume() and onStart()?

## Kotlin Programming (8 Questions)

1. What is null safety in Kotlin and how does it prevent NullPointerException?
2. What are data classes and what benefits do they provide?
3. Explain sealed classes in Kotlin and provide use cases
4. What are extension functions and when should you use them?
5. Explain scope functions (let, run, with, apply, also) with examples
6. What are coroutines and how do they differ from threads?
7. What are inline functions and when to use them?
8. Explain type system in Kotlin (Any, Nothing, Unit)

## Java vs Kotlin (10 Questions)

1. Why is Kotlin preferred over Java for Android development?
2. What are the main syntax differences between Java and Kotlin?
3. How does Kotlin handle null safety compared to Java?
4. Explain extension functions in Kotlin and how they compare to Java
5. What are data classes and how do they compare to Java POJOs?
6. How does Kotlin interop with Java?
7. Explain coroutines vs Java threads
8. What are sealed classes and how do they differ from enums?
9. How to migrate from Java to Kotlin?
10. Compare performance of Kotlin vs Java

## Android Architecture (12 Questions)

1. Explain MVVM architecture pattern
2. What is the difference between MVVM and MVP patterns?
3. Explain Clean Architecture in Android
4. What is the Repository pattern?
5. Explain SOLID principles with Android examples
6. What is dependency injection and why is it important?
7. Explain LiveData vs Flow
8. How does ViewModel work and when to use it?
9. Explain multi-module architecture
10. How to implement offline-first architecture?
11. What is the Single Source of Truth pattern?
12. Explain error handling in MVVM architecture

## Jetpack Compose (10 Questions)

1. What is Jetpack Compose and how does it differ from XML layouts?
2. Explain recomposition in Jetpack Compose
3. Explain state management in Compose (@State, @StateObject)
4. How to handle navigation in Jetpack Compose?
5. What is state hoisting in Compose?
6. How to optimize Compose performance?
7. Explain Compose lifecycle and side effects
8. How to test Compose UI?
9. Compare LazyColumn with RecyclerView
10. How to handle animations in Compose?

## MVI Pattern (8 Questions)

1. What is MVI (Model-View-Intent) pattern?
2. Explain unidirectional data flow in MVI
3. How to handle side effects in MVI?
4. What are sealed classes for state management in MVI?
5. How to test MVI architecture?
6. Compare MVI with MVVM
7. How is MVI similar to Redux?
8. When to use MVI pattern?

## Android KMM (8 Questions)

1. What is Android KMM (Kotlin Multiplatform Mobile)?
2. How does code sharing work in KMM?
3. Explain expect/actual mechanism in KMM
4. What are the limitations of KMM?
5. Compare KMM with Flutter
6. How to handle platform-specific code in KMM?
7. What is the use case for KMM?
8. How to structure a KMM project?

## Coroutines & Flows (10 Questions)

1. What are coroutines and how do they differ from RxJava?
2. Explain suspending functions
3. What are Dispatchers and when to use each?
4. How to handle coroutine cancellation?
5. Explain StateFlow vs SharedFlow
6. What are cold flows and hot flows?
7. When to use viewModelScope vs GlobalScope?
8. How to test coroutines?
9. Explain race conditions in coroutines
10. How to use WorkManager with coroutines?

## Data Persistence (8 Questions)

1. Compare Room, SQLite, and Realm
2. How to handle database migrations in Room?
3. Compare DataStore with SharedPreferences
4. How to implement offline-first architecture?
5. How to optimize Room database performance?
6. What are Room transactions?
7. How to encrypt Room database?
8. Explain Room queries and operations

## Networking (8 Questions)

1. Compare Retrofit, Volley, and Ktor
2. How to handle errors in Retrofit?
3. Explain SSL Pinning implementation
4. How to implement caching strategies?
5. Explain OkHttp interceptors
6. How to implement retry logic with exponential backoff?
7. How to handle authentication tokens?
8. How to handle file uploads/downloads with progress?

## Dependency Injection (8 Questions)

1. What is dependency injection and why is it important?
2. Compare Dagger, Hilt, and Koin
3. Explain DI scopes (Singleton, Activity, ViewModel)
4. What is the difference between constructor and field injection?
5. How to test applications with dependency injection?
6. What are qualifiers in DI?
7. How to inject ViewModels?
8. Explain Hilt setup and usage

## Memory Management (8 Questions)

1. Explain Android memory model
2. How does garbage collection work on Android?
3. How to prevent memory leaks?
4. How to optimize bitmap memory usage?
5. How does LeakCanary work?
6. What tools to use for memory profiling?
7. Explain WeakReference, SoftReference, and PhantomReference
8. How to profile memory in Android?

## Testing (7 Questions)

1. What are unit, integration, and UI tests?
2. How to test ViewModels?
3. What are test doubles (mocks, stubs, fakes)?
4. How to use MockK for mocking?
5. How to write Espresso tests?
6. How to test coroutines in tests?
7. Explain test-driven development (TDD)

## Gradle & Maven (8 Questions)

1. Compare Gradle and Maven
2. What is the difference between implementation and api?
3. How to configure build variants and flavors?
4. Compare ProGuard and R8
5. How to optimize Gradle build performance?
6. How to create custom Gradle tasks?
7. Explain multi-module Gradle project structure
8. What are Gradle configurations?

## Publishing to Play Store (9 Questions)

1. Compare Personal vs Organization Play Store accounts
2. What is the complete publishing process to Play Store?
3. Explain release tracks (internal, alpha, beta, production)
4. What is the difference between AAB and APK?
5. How to sign an Android app?
6. Explain staged rollout strategy
7. What are common app rejection reasons?
8. How to set up in-app products?
9. Explain app versioning and updates

## Security (8 Questions)

1. How to implement SSL Pinning?
2. How to secure sensitive data in Android app?
3. How to secure API keys?
4. How to prevent reverse engineering?
5. How to use ProGuard/R8 effectively?
6. How to implement encryption at rest?
7. Explain certificate pinning implementation
8. How to implement secure communication?

## Advanced Topics (8 Questions)

1. Explain Jetpack Architecture Components
2. How to structure multi-module architecture?
3. How to optimize app performance?
4. What are profiling tools for Android?
5. How to implement CI/CD for Android?
6. Explain all Jetpack libraries
7. How to implement Material Design 3?
8. Explain app architecture best practices

---

**Return to:** [Home](/)

