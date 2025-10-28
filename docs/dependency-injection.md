---
layout: default
title: "Dependency Injection - Interview Preparation"
---

# Dependency Injection

## Table of Contents
- [DI Concepts](#di-concepts)
- [Hilt](#hilt)
- [Koin](#koin)
- [Interview Questions & Answers](#interview-questions--answers)

## Hilt

```kotlin
@HiltAndroidApp
class MyApplication : Application()

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .build()
    }
    
    @Provides
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    
    @Inject
    lateinit var viewModel: MainViewModel
}
```

## Interview Questions & Answers

### Q1: What is dependency injection and why is it important?

**Answer:**

Dependency Injection provides dependencies to classes rather than having classes create them.

**Benefits:**
- Loose coupling
- Easier testing
- Better code organization
- Reusability

```kotlin
// Without DI
class ViewModel {
    private val repository = UserRepository() // Tight coupling
}

// With DI
class ViewModel(
    private val repository: UserRepository
) // Loose coupling, testable
```

### Q2: Compare Dagger, Hilt, and Koin

**Answer:**

**Conceptual Overview:**

These three DI frameworks solve the same problem but with different approaches. Understanding their differences helps choose the right tool for your project.

**Dagger:** The original, most powerful but complex. Low-level control with code generation.

**Hilt:** Built on top of Dagger, specifically for Android. Reduces boilerplate while maintaining Dagger's power.

**Koin:** Simple and pragmatic. Uses reflection at runtime instead of code generation at compile time.

**Key Trade-offs:**
- Performance: Code generation (Dagger/Hilt) vs Reflection (Koin)
- Learning Curve: Steep (Dagger) vs Medium (Hilt) vs Easy (Koin)
- Setup: Complex (Dagger/Hilt) vs Simple (Koin)
- Compile-time Safety: Yes (Dagger/Hilt) vs No (Koin)

| Feature | Dagger | Hilt | Koin |
|---------|--------|------|------|
| Setup | Complex | Simple | Simple |
| Code Generation | Yes | Yes | No (runtime) |
| Annotation Processing | Yes | Yes | No |
| Performance | Best | Best | Good |
| Learning Curve | Steep | Medium | Easy |

### Q3: Explain DI scopes (Singleton, Activity, ViewModel)

**Answer:**

**Conceptual Foundation:**

Scopes in dependency injection determine the lifetime of a dependency instance. Understanding scopes is crucial for managing memory and ensuring objects are available when needed without leaking.

**Scope Hierarchy:**
- Singleton: Lives for the entire app lifetime
- Activity: Lives while Activity exists
- ViewModel: Lives while ViewModel exists
- Fragment: Lives while Fragment exists

**Why Scopes Matter:**
- Memory management: Avoid keeping objects longer than needed
- State preservation: Keep objects alive as long as required
- Resource sharing: Share instances within a scope

```kotlin
// Singleton - One instance for entire app
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit { }
}

// Activity scope - One per activity
@Module
@InstallIn(ActivityComponent::class)
object ActivityModule {
    @Provides
    fun provideApiService(retrofit: Retrofit): ApiService { }
}

// ViewModel scope
@Module
@InstallIn(ViewModelComponent::class)
object ViewModelModule {
    @Provides
    fun provideRepository(): Repository { }
}
```

### Q4: Constructor vs Field injection

**Answer:**

**Conceptual Understanding:**

The choice between constructor and field injection affects code clarity, testability, and object initialization. Each approach has its use cases.

**Constructor Injection:**
- Dependencies are required parameters
- Objects are initialized with all dependencies
- Immutable dependencies (val)
- Easier to test
- Better for ViewModels and business logic

**Field Injection:**
- Dependencies set after object creation
- Requires lateinit var
- Mutable dependencies
- Necessary for framework classes (Activity, Fragment)
- Used when you can't control object construction

**Best Practice:**
- Use constructor injection when possible
- Use field injection only for framework classes

**Constructor Injection (Recommended):**
```kotlin
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() { }
```

**Field Injection:**
```kotlin
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var viewModel: UserViewModel
}
```

### Q5: How to test with DI?

**Answer:**

**Conceptual Understanding:**

DI makes testing easier by allowing you to replace real dependencies with test doubles (mocks, stubs, fakes). This enables isolated unit testing and predictable test behavior.

**Testing Benefits:**
- Replace slow/fragile dependencies (database, network)
- Control dependency behavior
- Test edge cases easily
- Fast, isolated tests

**Testing Strategies:**
- Mock dependencies for unit tests
- Provide test implementations
- Use test modules with Hilt
- Create test doubles manually

```kotlin
class UserViewModelTest {
    @Test
    fun testWithDI() {
        val mockRepository = mockk<UserRepository>()
        val viewModel = UserViewModel(mockRepository)
        
        // Test viewModel
    }
}
```

### Q6: What are qualifiers in DI?

**Answer:**

**Conceptual Understanding:**

Qualifiers are annotations that help DI frameworks distinguish between multiple implementations of the same type. When you have multiple instances or implementations of the same interface, qualifiers tell the framework which one to inject.

**Why Qualifiers Are Needed:**
- Multiple implementations of the same interface
- Different configurations (production vs test)
- Named instances (e.g., remote vs local)
- Environment-specific implementations

**How They Work:**
- Create a custom annotation with @Qualifier
- Apply it to implementations
- Apply it to injection sites
- Framework uses qualifier to resolve correct instance

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class RemoteDataSource

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class LocalDataSource

@Provides
@RemoteDataSource
fun provideRemoteDataSource(): DataSource = RemoteDataSourceImpl()

@Provides
@LocalDataSource
fun provideLocalDataSource(): DataSource = LocalDataSourceImpl()
```

### Q7: How to inject ViewModels?

**Answer:**

```kotlin
// With Hilt
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
}

// Manual DI
class UserViewModelFactory @Inject constructor(
    private val repository: UserRepository
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return UserViewModel(repository) as T
    }
}
```

### Q8: Explain Hilt setup and usage

**Answer:**

Hilt streamlines DI on Android with lifecycle-aware components and minimal boilerplate.
Annotate your Application, Android components, and modules; Hilt wires dependencies automatically.

**Conceptual Foundation:**

Hilt is a dependency injection library for Android that reduces boilerplate of manual DI setup. It's built on top of Dagger and specifically designed for Android applications.

**Why Hilt:**
- Simplified setup compared to Dagger
- Automatic component generation
- Built-in Android lifecycle awareness
- Less boilerplate code
- Easier testing support

**Key Concepts:**
- Annotation-based setup
- Component hierarchy aligned with Android lifecycle
- Automatic dependency injection into Activities and Fragments
- ViewModels integration with scope management

**Setup:**
```gradle
// build.gradle
plugins {
    id 'com.google.dagger.hilt.android' version '2.48'
}

// Application class
@HiltAndroidApp
class MyApplication : Application()

// Activity/Fragment
@AndroidEntryPoint
class MainActivity : AppCompatActivity()

// Module
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit { }
}
```

---

**Previous:** [Networking](networking.md)

**Next:** [Memory Management](memory-management.md)

