---
layout: default
title: "Advanced Topics - Interview Preparation"
---

# Advanced Topics

## Table of Contents
- [Jetpack Components](#jetpack-components)
- [Multi-Module Architecture](#multi-module-architecture)
- [Performance Optimization](#performance-optimization)
- [CI/CD](#cicd)
- [Interview Questions & Answers](#interview-questions--answers)

## Multi-Module Architecture

```
:app
:feature:user
:feature:post
:core:network
:core:database
:core:ui
```

## Interview Questions & Answers

### Q1: Explain Jetpack Architecture Components

**Answer:**

Jetpack is a suite of libraries for Android development.

**Key Components:**
- LiveData
- ViewModel
- Room
- Navigation
- WorkManager
- Data Binding
- Paging Library

```kotlin
// ViewModel
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    val users = repository.getUsers().asLiveData()
}

// Room
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase()

// Navigation
val navController = rememberNavController()
NavHost(navController, startDestination = "home")
```

### Q2: How to structure multi-module architecture?

**Answer:**

**Conceptual Understanding:**

Multi-module architecture organizes code into separate Gradle modules, each with distinct responsibilities. This approach follows the Single Responsibility Principle at a module level, making the codebase more maintainable and scalable.

**Benefits:**
- Faster incremental builds (only changed modules rebuild)
- Better encapsulation (module boundaries)
- Reusable modules across projects
- Parallel work by different team members
- Independent testing and deployment

**Design Principles:**
- Each module has a clear purpose
- Dependencies flow in one direction
- Core modules are stable, feature modules depend on them
- No circular dependencies

Multi-module architecture splits the app into separate Gradle modules for better organization and build times.

**Module Hierarchy:**
```
:app (main module)
  └── :feature:auth
  └── :feature:home
  └── :feature:profile
  └── :feature:settings
  └── :core:network
  └── :core:database
  └── :core:ui
  └── :core:common
```

**Benefits:**
- Faster incremental builds
- Better separation of concerns
- Reusable modules
- Independent testing

```kotlin
// feature:auth module
@AndroidEntryPoint
class LoginFragment : Fragment() {
    private val viewModel: LoginViewModel by viewModels()
}

class LoginViewModel @Inject constructor(
    private val authRepository: AuthRepository
) : ViewModel() { }

// feature:home module
class HomeViewModel @Inject constructor(
    private val homeRepository: HomeRepository
) : ViewModel() { }

// :core:network module (shared)
class ApiModule {
    @Provides
    fun provideRetrofit(): Retrofit { }
}
```

### Q3: How to optimize app performance?

**Answer:**

**Performance Philosophy:**

Optimizing app performance is about making your app fast, responsive, and resource-efficient. Key areas include:
- Reducing memory allocation
- Minimizing UI work on main thread
- Efficient data structures
- Lazy loading and caching
- Background processing

**Critical Metrics:**
- App startup time
- UI rendering (60 fps)
- Memory usage
- Battery consumption
- Network efficiency

**Performance Optimization Strategies:**

**1. Lazy Loading:**
```kotlin
@Composable
fun LazyUserList(users: List<User>) {
    LazyColumn {
        items(users) { user ->
            UserItem(user) // Items load as needed
        }
    }
}
```

**2. View Recycling:**
```kotlin
// Use stable keys
LazyColumn {
    items(items, key = { it.id }) { item ->
        ItemCard(item)
    }
}
```

**3. Background Threading:**
```kotlin
viewModelScope.launch(Dispatchers.IO) {
    val data = heavyComputation()
    withContext(Dispatchers.Main) {
        updateUI(data)
    }
}
```

### Q4: What are profiling tools for Android?

**Answer:**

**Conceptual Understanding:**

Profiling tools help developers identify performance bottlenecks, memory issues, and optimization opportunities in Android apps. Different tools serve different purposes in the performance optimization workflow.

**Profiling Goals:**
- Identify CPU bottlenecks
- Detect memory leaks
- Optimize rendering performance
- Analyze network usage
- Measure battery impact

**Tool Categories:**
1. Integrated profilers (Android Studio)
2. Automated leak detection (LeakCanary)
3. System-level profilers (Systrace, Perfetto)
4. Hardware profilers

**Android Studio Profiler:**
- CPU Profiler: Identify bottlenecks
- Memory Profiler: Detect leaks
- Network Profiler: Monitor network usage
- Energy Profiler: Battery impact

**LeakCanary:**
```gradle
debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.12'
```

**Systrace:**
```bash
python systrace.py --time=10 -o trace.html sched
```

### Q5: How to implement CI/CD for Android?

**Answer:**

**Conceptual Foundation:**

CI/CD (Continuous Integration/Continuous Deployment) automates the process of building, testing, and deploying Android apps. This ensures code quality and reduces manual errors.

**CI/CD Benefits:**
- Automatic testing on every commit
- Fast feedback on code changes
- Consistent builds across environments
- Automated deployment to app stores
- Better collaboration in teams

**CI/CD Pipeline Stages:**
1. Build: Compile code and create APK/AAB
2. Test: Run unit, integration, and UI tests
3. Analyze: Code quality checks, linting
4. Deploy: Upload to Play Store or distribution channels

**GitHub Actions Example:**
```yaml
name: Build and Test

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-java@v3
      with:
        java-version: '17'
    - name: Grant execute permission
      run: chmod +x gradlew
    - name: Build
      run: ./gradlew build
    - name: Run tests
      run: ./gradlew test
```

### Q6: Explain all Jetpack libraries

**Answer:**

**Conceptual Overview:**

Jetpack is a suite of libraries, tools, and guidance to help build high-quality Android apps. It's designed to make common tasks easier and provide architectural guidance.

**Jetpack Categories:**
- Architecture Components: MVVM, Navigation, Room
- Foundation: Core, AppCompat, Multidex
- UI: Compose, Material, Window Manager
- Behavior: WorkManager, Notifications, Permissions

**Why Jetpack:**
- Accelerate development
- Handle boilerplate code
- Best practices built-in
- Backwards compatibility
- Google support and updates

**Architecture Libraries:**
- **Navigation**: Manage navigation
- **Room**: Database abstraction
- **ViewModel**: UI data management
- **LiveData**: Observable data holder
- **DataStore**: Async key-value storage

**UI Libraries:**
- **Compose**: Modern UI toolkit
- **Material**: Material Design components

**Foundation Libraries:**
- **Core**: Backward compatibility
- **AppCompat**: App compatibility
- **Fragment**: Fragment support

### Q7: How to implement Material Design 3?

**Answer:**

**Conceptual Understanding:**

Material Design 3 is the latest evolution of Google's design system, providing dynamic color schemes, updated components, and improved personalization features.

**Key Features:**
- Dynamic color theming
- Enhanced Material You support
- Updated component styles
- Dark mode improvements
- Better accessibility

**Implementation Areas:**
- Color system and theming
- Typography and fonts
- Component styling
- Animation and motion
- Responsive layouts

**Color Scheme:**
```kotlin
val darkColorScheme = darkColorScheme(
    primary = Purple80,
    secondary = PurpleGrey80
)

val lightColorScheme = lightColorScheme(
    primary = Purple40,
    secondary = PurpleGrey40
)

@Composable
fun MyApp() {
    val colorScheme = if (isDarkTheme) darkColorScheme else lightColorScheme
    
    MaterialTheme(colorScheme = colorScheme) {
        MyScreen()
    }
}
```

### Q8: Explain app architecture best practices

**Answer:**

Adopt a layered architecture with clear boundaries and dependencies flowing inward.
Keep UI logic in ViewModels, business rules in use cases, and data access behind repositories.

**Layered Architecture:**
```
Presentation Layer (UI + ViewModels)
    ↓
Domain Layer (Use Cases)
    ↓
Data Layer (Repository + Data Sources)
```

**Principles:**
- Single Responsibility
- Dependency Rule
- Use Cases for business logic
- Repository pattern for data
- ViewModels for UI state

---

**Previous:** [Security](security.md)

**Next:** [Home](/)

