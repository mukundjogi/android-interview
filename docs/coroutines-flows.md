---
layout: default
title: "Coroutines & Flows - Interview Preparation"
---

# Coroutines & Flows

## Table of Contents
- [Coroutines Basics](#coroutines-basics)
- [Dispatchers](#dispatchers)
- [Flows](#flows)
- [StateFlow vs SharedFlow](#stateflow-vs-sharedflow)
- [Interview Questions & Answers](#interview-questions--answers)

## Coroutines

```kotlin
class MainViewModel : ViewModel() {
    
    fun loadData() {
        viewModelScope.launch(Dispatchers.IO) {
            val data = fetchData()
            withContext(Dispatchers.Main) {
                updateUI(data)
            }
        }
    }
    
    private suspend fun fetchData(): Data {
        // Network call
        return apiService.getData()
    }
}
```

## Flows

```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    
    val users: Flow<List<User>> = repository.getAllUsers()
        .catch { e -> emit(emptyList()) }
        .flowOn(Dispatchers.IO)
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
}
```

## Coroutines Basics

**What are Coroutines?**

Coroutines are Kotlin's modern way of handling asynchronous programming. Think of them as lightweight workers that can pause their work, do something else, and then come back to finish their task without blocking the main thread.

**Why Coroutines Matter:**

- **Non-blocking**: Don't freeze your app while waiting for data
- **Lightweight**: Thousands of coroutines can run on a single thread
- **Sequential Code**: Write async code that looks like regular code
- **Structured**: Easy to manage, cancel, and handle errors
- **Efficient**: Much more efficient than traditional threads

**Key Concepts:**

1. **Suspend Functions**: Functions that can pause execution
2. **Coroutine Scope**: Defines the lifetime of coroutines
3. **Dispatchers**: Determine which thread coroutines run on
4. **Structured Concurrency**: Parent coroutines control child coroutines

**How Coroutines Work:**

```kotlin
class MainViewModel : ViewModel() {
    
    fun loadData() {
        viewModelScope.launch(Dispatchers.IO) {
            val data = fetchData()
            withContext(Dispatchers.Main) {
                updateUI(data)
            }
        }
    }
    
    private suspend fun fetchData(): Data {
        // Network call - this won't block the UI
        return apiService.getData()
    }
}
```

**What happens here:**
1. `viewModelScope.launch` starts a coroutine
2. `Dispatchers.IO` runs the coroutine on a background thread
3. `fetchData()` is a suspend function that can pause
4. `withContext(Dispatchers.Main)` switches back to UI thread
5. UI stays responsive throughout the process

## Dispatchers

**What are Dispatchers?**

Dispatchers are like traffic controllers that decide which thread pool your coroutines run on. Think of them as different lanes on a highway - each lane is designed for different types of traffic.

**Why Dispatchers Matter:**

- **Thread Management**: Automatically choose the right thread
- **Performance**: Optimize for different types of work
- **UI Safety**: Ensure UI updates happen on the main thread
- **Resource Management**: Efficiently use system resources

**The Main Dispatchers:**

1. **Main** - The "UI Lane"
   - Runs on the main/UI thread
   - Use for UI updates, user interactions
   - Only one thread, so don't do heavy work here

2. **IO** - The "Network Lane"
   - Designed for I/O operations
   - Use for network calls, database operations, file I/O
   - Can handle many concurrent operations

3. **Default** - The "CPU Lane"
   - Designed for CPU-intensive work
   - Use for calculations, data processing, sorting
   - Limited number of threads (usually CPU cores)

4. **Unconfined** - The "Wild Lane"
   - No specific thread requirement
   - Not recommended for production code
   - Mainly for testing

```kotlin
// Main - UI thread
viewModelScope.launch(Dispatchers.Main) {
    textView.text = "Updated" // UI operations
}

// IO - Background thread for I/O
viewModelScope.launch(Dispatchers.IO) {
    val data = api.getData() // Network call
    database.save(data) // Database operation
}

// Default - Background thread for CPU work
viewModelScope.launch(Dispatchers.Default) {
    val result = heavyComputation() // CPU-intensive work
}

// Custom dispatcher
val customDispatcher = Executors.newFixedThreadPool(4).asCoroutineDispatcher()
```

**When to Use Each:**

- **Main**: UI updates, user interactions, animations
- **IO**: Network calls, database operations, file I/O
- **Default**: Calculations, data processing, image processing
- **Custom**: When you need specific thread pool behavior

## Flows

**What are Flows?**

Flows are Kotlin's way of handling streams of data over time. Think of them as a river of data that flows from a source to wherever it's needed, with the ability to transform, filter, and combine the data along the way.

**Why Flows Matter:**

- **Reactive Programming**: Automatically update UI when data changes
- **Backpressure**: Handle data faster than it can be processed
- **Composition**: Combine multiple data sources easily
- **Cancellation**: Automatically stop when no longer needed

**Key Concepts:**

1. **Cold Flows**: Start when collected, independent for each collector
2. **Hot Flows**: Emit data regardless of collectors (StateFlow, SharedFlow)
3. **Operators**: Transform, filter, and combine data
4. **Collectors**: Consume the data stream

**How Flows Work:**

```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    
    val users: Flow<List<User>> = repository.getAllUsers()
        .catch { e -> emit(emptyList()) } // Handle errors
        .flowOn(Dispatchers.IO) // Run on IO thread
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
}
```

**What happens here:**
1. `repository.getAllUsers()` returns a Flow of users
2. `.catch` handles any errors gracefully
3. `.flowOn(Dispatchers.IO)` runs the flow on background thread
4. `.stateIn` converts to StateFlow for UI consumption
5. `WhileSubscribed(5000)` keeps data for 5 seconds after last collector

## StateFlow vs SharedFlow

**What's the Difference?**

Both StateFlow and SharedFlow are "hot" flows that emit data even when no one is listening, but they serve different purposes.

**StateFlow - The "Current State" Flow:**

- **Always has a value**: Represents the current state
- **New collectors get current value**: Immediately see the latest state
- **Perfect for UI state**: User profile, settings, current screen
- **Single source of truth**: One authoritative value

**SharedFlow - The "Event Stream" Flow:**

- **May not have a value**: Represents events that happen
- **New collectors don't get past events**: Only see new events
- **Perfect for events**: Navigation, toasts, one-time notifications
- **Multiple events**: Can emit many events over time

**When to Use Each:**

```kotlin
// StateFlow - for state that should persist
class UserViewModel : ViewModel() {
    
    private val _state = MutableStateFlow<UserState>(UserState.Idle)
    val state: StateFlow<UserState> = _state.asStateFlow()
    
    // Always has current value
    val currentValue = _state.value
}

// SharedFlow - for events that shouldn't be replayed
class EventBus : ViewModel() {
    
    private val _events = MutableSharedFlow<Event>(
        replay = 1, // Keep last 1 event
        extraBufferCapacity = 1
    )
    val events: SharedFlow<Event> = _events.asSharedFlow()
    
    fun emit(event: Event) {
        _events.emit(event)
    }
}
```

**Use StateFlow for:**
- User profile data
- App settings
- Current screen state
- Loading states

**Use SharedFlow for:**
- Navigation events
- Toast messages
- One-time notifications
- User actions

## Interview Questions & Answers

### Q1: What are coroutines and how do they differ from RxJava?

**Answer:**

**Theoretical Foundation:**

Coroutines are Kotlin's native solution for handling asynchronous programming. They represent a paradigm shift from callback-based or reactive stream approaches to a more sequential, thread-like coding style that's easier to read and reason about.

**Conceptual Understanding:**

Coroutines are a form of cooperative multitasking. Unlike threads (which are preempted by the OS), coroutines voluntarily yield execution, allowing other coroutines to run. This makes them lightweight and efficient.

**Key Theoretical Differences from RxJava:**

1. **Programming Model**:
   - **Coroutines**: Sequential code that looks synchronous but is actually asynchronous
   - **RxJava**: Functional reactive programming with operators and streams

2. **Concurrency Model**:
   - **Coroutines**: Structured concurrency - parent scopes control child lifecycle
   - **RxJava**: Subscription-based - manual lifecycle management required

3. **Learning Curve**:
   - **Coroutines**: Easier to understand for developers familiar with threads/synchronous code
   - **RxJava**: Steeper learning curve with functional programming concepts

4. **Error Handling**:
   - **Coroutines**: Traditional try-catch works across coroutines
   - **RxJava**: Reactive error handling with onError callbacks

5. **Cancellation**:
   - **Coroutines**: Automatic cancellation propagation through structured concurrency
   - **RxJava**: Manual unsubscribe required, no automatic propagation

**Why Coroutines Matter:**

- **Simpler Mental Model**: Code reads top-to-bottom like synchronous code
- **Better Integration**: Works seamlessly with existing Kotlin features
- **Structured Concurrency**: Prevents leaks automatically through scope management
- **Performance**: Hundreds of coroutines can run on a single thread
- **Cancellation Safety**: Built-in cancellation throughout the execution flow

**Differences:**
- Lightweight vs Heavy: One coroutine uses only a few KB vs ~1MB per thread
- Structured concurrency: Automatic lifecycle management
- Built-in cancellation: Exception-safe cancellation propagation
- No subscription management needed: No need to track and unsubscribe
- Synchronous-looking code: Reads like sequential code

```kotlin
// Coroutines
viewModelScope.launch {
    val user = fetchUser()
    updateUI(user)
}

// RxJava (for comparison)
Observable.fromCallable { fetchUser() }
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe { user -> updateUI(user) }
```

### Q2: Explain suspending functions

**Answer:**

Suspending functions can suspend execution without blocking threads.

```kotlin
// Suspending function
suspend fun fetchUser(userId: String): User {
    return withContext(Dispatchers.IO) {
        api.getUser(userId)
    }
}

// Call from coroutine
viewModelScope.launch {
    val user = fetchUser("123") // Suspends here, doesn't block
    updateUI(user)
}

// Chain suspending functions
suspend fun fetchAndProcess(userId: String): ProcessedUser {
    val user = fetchUser(userId) // Suspend
    return processUser(user) // Suspend
}

suspend fun processUser(user: User): ProcessedUser {
    delay(100) // Simulate work
    return ProcessedUser(user)
}
```

### Q3: What are Dispatchers and when to use each?

**Answer:**

**Theoretical Concept:**

Dispatchers determine which thread pool a coroutine runs on. They're essentially thread managers that provide the execution context for coroutines. Understanding dispatchers is crucial for writing performant concurrent code.

**Why Dispatchers Matter:**

Dispatchers abstract away thread management. Instead of manually choosing threads, you choose a dispatcher based on the type of work you're doing. This ensures optimal thread pool usage and prevents common concurrency issues.

**Core Dispatchers Explained:**

**When to Use Each:**

```kotlin
// Main - UI thread
viewModelScope.launch(Dispatchers.Main) {
    textView.text = "Updated" // UI operations
}

// IO - Network, database
viewModelScope.launch(Dispatchers.IO) {
    val data = api.getData() // Network call
    database.save(data) // Database operation
}

// Default - CPU-intensive work
viewModelScope.launch(Dispatchers.Default) {
    val result = heavyComputation() // Computation
}

// Unconfined - No specific thread
viewModelScope.launch(Dispatchers.Unconfined) {
    // Not recommended for production
}

// Custom dispatcher
val customDispatcher = Executors.newFixedThreadPool(4).asCoroutineDispatcher()
```

### Q4: How to handle coroutine cancellation?

**Answer:**

```kotlin
class DataLoader : ViewModel() {
    
    fun loadData() {
        viewModelScope.launch {
            try {
                val data = fetchData() // Check cancellation internally
                processData(data)
            } catch (e: CancellationException) {
                // Handle cancellation
                cleanup()
                throw e
            }
        }
    }
    
    private suspend fun fetchData(): Data {
        return withContext(Dispatchers.IO) {
            // Check cancellation periodically
            ensureActive()
            api.getData()
        }
    }
}

// Job cancellation
var job: Job? = null

fun startLoading() {
    job = viewModelScope.launch {
        loadData()
    }
}

fun stopLoading() {
    job?.cancel()
}

// Handle cancellation
suspend fun processItems(items: List<Item>) {
    for (item in items) {
        ensureActive() // Check if cancelled
        processItem(item)
    }
}
```

### Q5: Explain StateFlow vs SharedFlow

**Answer:**

**Theoretical Concepts:**

Both StateFlow and SharedFlow are hot streams (they emit even when no collectors are listening). The key difference is in their semantics and use cases. Understanding when to use each is crucial for proper state management.

**StateFlow:**
- Designed to hold state (always has a current value)
- Always emits the current value to new collectors
- Used for representing and sharing UI state
- Similar to LiveData but with better Kotlin integration

**SharedFlow:**
- Designed for events (may not have a current value)
- New collectors don't necessarily get past emissions
- Used for one-time events, notifications, UI events
- More flexible than StateFlow for event streams

**When to Use StateFlow:**
- UI state that should persist (user profile, settings, etc.)
- State that new observers should immediately see
- When you need a single source of truth

**When to Use SharedFlow:**
- Events that shouldn't be replayed (navigation events, toasts)
- One-time notifications
- Events that new collectors shouldn't see

**StateFlow - Hot flow with single value:**
```kotlin
class UserViewModel : ViewModel() {
    
    private val _state = MutableStateFlow<UserState>(UserState.Idle)
    val state: StateFlow<UserState> = _state.asStateFlow()
    
    // Always has current value
    val currentValue = _state.value
}

// Collector always receives latest value
viewModel.state.collect { state ->
    // Receives current state immediately
}
```

**SharedFlow - Hot flow with replay:**
```kotlin
class EventBus : ViewModel() {
    
    private val _events = MutableSharedFlow<Event>(
        replay = 1,
        extraBufferCapacity = 1
    )
    val events: SharedFlow<Event> = _events.asSharedFlow()
    
    fun emit(event: Event) {
        _events.emit(event)
    }
}

// Collect events
events.collect { event ->
    // Handle event
}
```

**Key Differences:**
- StateFlow: Always has value, collectors get current value
- SharedFlow: May not have value, collectors don't get current by default
- Use StateFlow for state
- Use SharedFlow for events

### Q6: What are cold flows and hot flows?

**Answer:**

**Cold Flow - Starts when collected:**
```kotlin
fun getUsers(): Flow<List<User>> = flow {
    println("Flow started") // Executes on each collection
    emit(listOf(User("1", "John")))
}

// Each collector gets independent emissions
getUsers().collect { users -> println(users) }
getUsers().collect { users -> println(users) } // Starts again
```

**Hot Flow - Shared among collectors:**
```kotlin
val hotFlow = MutableSharedFlow<Int>()

// Shared among collectors
hotFlow.collect { value -> println("Collector 1: $value") }
hotFlow.collect { value -> println("Collector 2: $value") }

hotFlow.emit(1) // Both collectors receive
```

### Q7: When to use viewModelScope vs GlobalScope?

**Answer:**

**viewModelScope (Recommended):**
```kotlin
class UserViewModel : ViewModel() {
    
    fun loadData() {
        viewModelScope.launch {
            // Automatically cancelled when ViewModel cleared
            val data = fetchData()
        }
    }
}
```
- Automatically cancelled
- Lifecycle-aware
- Tied to ViewModel lifecycle

**GlobalScope (Avoid in production):**
```kotlin
class BadExample {
    fun loadData() {
        GlobalScope.launch {
            // Runs for entire app lifetime
            // May cause memory leaks
        }
    }
}
```
- Not automatically cancelled
- Not lifecycle-aware
- May cause memory leaks

**Exception - Background work:**
```kotlin
class WorkManagerExample : CoroutineWorker(context, params) {
    override suspend fun doWork(): Result {
        // WorkManager manages scope
        fetchData()
        return Result.success()
    }
}
```

### Q8: How to test coroutines?

**Answer:**

```kotlin
class UserViewModelTest {
    
    @get:Rule
    val mainCoroutineRule = MainCoroutineRule()
    
    @Test
    fun `load users updates state`() = runTest {
        val repository = mockk<UserRepository>()
        coEvery { repository.getUsers() } returns listOf(User("1", "John"))
        
        val viewModel = UserViewModel(repository)
        
        viewModel.loadUsers()
        
        advanceUntilIdle()
        
        val state = viewModel.state.value
        assertTrue(state is UiState.Success)
    }
}

class MainCoroutineRule : TestWatcher() {
    val testDispatcher = StandardTestDispatcher()
    
    override fun starting(description: Description) {
        Dispatchers.setMain(testDispatcher)
    }
    
    override fun finished(description: Description) {
        Dispatchers.resetMain()
        testDispatcher.cleanupTestCoroutines()
    }
}
```

### Q9: Explain race conditions in coroutines

**Answer:**

**Race Condition Problem:**
```kotlin
var counter = 0

fun incrementCounter() {
    viewModelScope.launch(Dispatchers.Default) {
        counter++ // Not thread-safe
    }
}

// Multiple concurrent calls may lose updates
repeat(100) { 
    incrementCounter() 
}
```

**Solution 1 - Mutex:**
```kotlin
var counter = 0
private val mutex = Mutex()

suspend fun incrementCounter() {
    mutex.withLock {
        counter++
    }
}
```

**Solution 2 - Atomic:**
```kotlin
var counter = AtomicInteger(0)

fun incrementCounter() {
    viewModelScope.launch {
        counter.incrementAndGet()
    }
}
```

**Solution 3 - Actor:**
```kotlin
sealed class CounterMessage
object Increment : CounterMessage()
class GetCounter(val response: CompletableDefer<Int>) : CounterMessage()

fun createCounter() = actor<CounterMessage> {
    var counter = 0
    for (msg in channel) {
        when (msg) {
            is Increment -> counter++
            is GetCounter -> msg.response.complete(counter)
        }
    }
}
```

### Q10: How to use WorkManager with coroutines?

**Answer:**

```kotlin
class UploadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            val fileUri = inputData.getString(KEY_FILE_URI) ?: return Result.failure()
            uploadFile(fileUri)
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
    
    private suspend fun uploadFile(uri: String) {
        // Upload logic
    }
}

// Enqueue work
val request = OneTimeWorkRequestBuilder<UploadWorker>()
    .setInputData(workDataOf(KEY_FILE_URI to fileUri))
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()
    )
    .build()

WorkManager.getInstance(context).enqueue(request)
```

---

**Previous:** [Android KMM](android-kmm.md)

**Next:** [Data Persistence](data-persistence.md)

