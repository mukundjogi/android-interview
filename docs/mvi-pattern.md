---
layout: default
title: "MVI Pattern - Interview Preparation"
---

# MVI Pattern

## Table of Contents
- [Overview](#overview)
- [Unidirectional Data Flow](#unidirectional-data-flow)
- [State Management](#state-management)
- [Interview Questions & Answers](#interview-questions--answers)

## Unidirectional Data Flow

```kotlin
sealed class Intent {
    object LoadPosts : Intent()
    data class LikePost(val postId: String) : Intent()
}

sealed class State {
    object Idle : State()
    object Loading : State()
    data class Success(val posts: List<Post>) : State()
    data class Error(val message: String) : State()
}

class PostViewModel : ViewModel() {
    private val _state = MutableStateFlow<State>(State.Idle)
    val state: StateFlow<State> = _state.asStateFlow()
    
    fun processIntent(intent: Intent) {
        when (intent) {
            is Intent.LoadPosts -> loadPosts()
            is Intent.LikePost -> likePost(intent.postId)
        }
    }
}
```

## Overview

**What is MVI (Model-View-Intent)?**

MVI is an architectural pattern that helps you organize your Android app code in a predictable way. Think of it like a well-organized restaurant where everyone has a clear role and things flow in one direction.

**Why Use MVI?**

- **Predictable**: You always know where data comes from and where it goes
- **Easy to Debug**: You can trace exactly why the UI changed
- **Testable**: Easy to test each part separately
- **Maintainable**: Clear separation of concerns

**The Three Main Components:**

1. **Model** - The "Kitchen" (Business Logic)
   - Contains your data and business rules
   - Processes intents and produces new states
   - Doesn't know about UI

2. **View** - The "Dining Room" (UI)
   - Shows the current state to the user
   - Sends intents when user interacts
   - Only displays what the model tells it to

3. **Intent** - The "Order" (User Actions)
   - Represents what the user wants to do
   - Like clicking a button, typing text, etc.
   - Sent from View to Model

**How MVI Works:**

```
User Action → Intent → Model → New State → View Updates
```

It's like this:
1. User clicks a button (Action)
2. View creates an Intent (Order)
3. Model processes the Intent (Kitchen prepares food)
4. Model produces new State (Food is ready)
5. View shows the new State (Food is served)

## Unidirectional Data Flow

**What is Unidirectional Data Flow?**

Unidirectional data flow means data moves in only one direction through your app. Think of it like a river - water only flows downstream, never upstream.

**Why Unidirectional Flow Matters:**

- **Predictable**: You always know where data comes from
- **Debuggable**: Easy to trace why something changed
- **Testable**: Each step can be tested independently
- **Maintainable**: Changes don't affect unexpected parts

**The Flow Cycle:**

```
User Action → Intent → ViewModel → State → View → (repeat)
```

**Detailed Explanation:**

1. **User Action**: User does something (clicks, types, swipes)
2. **Intent Creation**: View creates an Intent describing the action
3. **Intent Processing**: ViewModel receives and processes the Intent
4. **State Update**: ViewModel produces a new State based on the Intent
5. **UI Update**: View observes the new State and updates the UI

**Key Properties:**

- **Circular but Unidirectional**: Forms a circle but data never goes backwards
- **No Sideways Communication**: Views don't talk to each other directly
- **Single Authority**: Only ViewModel can change state
- **Predictable**: Every state change comes from an intent

```kotlin
sealed class Intent {
    object LoadPosts : Intent()
    data class LikePost(val postId: String) : Intent()
}

sealed class State {
    object Idle : State()
    object Loading : State()
    data class Success(val posts: List<Post>) : State()
    data class Error(val message: String) : State()
}

class PostViewModel : ViewModel() {
    private val _state = MutableStateFlow<State>(State.Idle)
    val state: StateFlow<State> = _state.asStateFlow()
    
    fun processIntent(intent: Intent) {
        when (intent) {
            is Intent.LoadPosts -> loadPosts()
            is Intent.LikePost -> likePost(intent.postId)
        }
    }
}
```

**How it works:**
1. User clicks "Load Posts" button
2. View creates `Intent.LoadPosts`
3. ViewModel processes the intent and calls `loadPosts()`
4. ViewModel updates state to `State.Loading`
5. View sees the loading state and shows a spinner
6. When posts load, ViewModel updates state to `State.Success(posts)`
7. View sees the success state and shows the posts

## State Management

**What is State Management in MVI?**

State management is how you handle and organize all the data that represents your app's current condition. Think of state as a snapshot of your app at any moment.

**Why State Management is Important:**

- **Single Source of Truth**: One place where all UI data lives
- **Predictable Updates**: Always know how state changes
- **Easy Testing**: Can test state transitions
- **Debugging**: Can see exactly what state caused what UI

**Key Concepts:**

1. **Immutable State**: State objects never change, you create new ones
2. **Single State Object**: All UI data in one place
3. **State Transitions**: Clear path from one state to another
4. **State Validation**: Ensure state is always valid

**State Design Principles:**

- **Complete**: State should contain everything the UI needs
- **Immutable**: Never modify state directly
- **Serializable**: Can be saved/restored
- **Testable**: Easy to create and verify

```kotlin
// Good state design
sealed class UserState {
    object Idle : UserState()
    object Loading : UserState()
    data class Success(
        val users: List<User>,
        val searchQuery: String = "",
        val selectedUser: User? = null
    ) : UserState()
    data class Error(
        val message: String,
        val retry: () -> Unit
    ) : UserState()
}

// Bad state design - multiple separate states
class BadUserViewModel {
    val isLoading = MutableStateFlow(false)
    val users = MutableStateFlow<List<User>>(emptyList())
    val error = MutableStateFlow<String?>(null)
    val searchQuery = MutableStateFlow("")
    // Hard to keep in sync!
}
```

**Benefits of Single State:**

- **Consistency**: All UI data is always in sync
- **Debugging**: One place to look for all state
- **Testing**: Easy to test complete state transitions
- **Predictability**: Always know what the UI will show

## Interview Questions & Answers

### Q1: What is MVI (Model-View-Intent) pattern?

**Answer:**

**Theoretical Foundation:**

MVI (Model-View-Intent) is an architectural pattern that enforces strict unidirectional data flow, popularized by concepts from React/Redux and inspired by functional reactive programming. It's an evolution of MVVM that adds more structure and predictability to state management.

**Core Philosophy:**

MVI is based on the principle that the UI is a pure function of state: `UI = f(state)`. Given the same state, you always get the same UI. This makes the UI predictable, testable, and debuggable.

**Key Concepts:**

1. **Single Source of Truth**: The State object is the only source of truth. Everything the UI displays comes from this state.

2. **Immutability**: State objects are immutable. When state needs to change, a new state object is created.

3. **Functional Programming**: The pattern treats the ViewModel as a reducer function that takes an Intent and current State, and produces a new State.

4. **Event Sourcing**: All user interactions are captured as Intent objects, creating an audit trail of user actions.

**Architectural Components:**

- **Intent**: Represents user actions/events. These are immutable objects that describe what the user wants to do.
- **State**: Represents the complete UI state. It's an immutable data structure that completely describes the UI.
- **Model**: The business logic layer that processes intents and produces new states.
- **View**: Observes state and emits intents. The view is completely reactive to state changes.

**Benefits Over MVVM:**
- More predictable state changes
- Easier debugging (can replay user actions)
- Better testability (test state transitions)
- Handles complex UI state more elegantly
- Exhaustive state handling with sealed classes

MVI enforces unidirectional data flow where:
- **Intent**: User actions (what user wants to do)
- **State**: UI state representation (complete UI description)
- **View**: Observes state and emits intents (pure rendering function)

```kotlin
// Intent - User actions
sealed class UserIntent {
    object LoadUsers : UserIntent()
    data class DeleteUser(val userId: String) : UserIntent()
    data class SearchUser(val query: String) : UserIntent()
}

// State - UI representation
sealed class UserState {
    object Idle : UserState()
    object Loading : UserState()
    data class Success(val users: List<User>) : UserState()
    data class Error(val message: String) : UserState()
}

// ViewModel processes intents
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    
    private val _intent = Channel<UserIntent>(Channel.UNLIMITED)
    private val _state = MutableStateFlow<UserState>(UserState.Idle)
    val state: StateFlow<UserState> = _state.asStateFlow()
    
    init {
        processIntents()
    }
    
    fun processIntent(intent: UserIntent) {
        _intent.trySend(intent)
    }
    
    private fun processIntents() {
        viewModelScope.launch {
            for (intent in _intent.receiveAsFlow()) {
                when (intent) {
                    is UserIntent.LoadUsers -> loadUsers()
                    is UserIntent.DeleteUser -> deleteUser(intent.userId)
                    is UserIntent.SearchUser -> searchUsers(intent.query)
                }
            }
        }
    }
    
    private fun loadUsers() {
        viewModelScope.launch {
            _state.value = UserState.Loading
            try {
                val users = repository.getUsers()
                _state.value = UserState.Success(users)
            } catch (e: Exception) {
                _state.value = UserState.Error(e.message ?: "Error")
            }
        }
    }
}

// View - Observes state
@Composable
fun UserScreen(viewModel: UserViewModel) {
    val state by viewModel.state.collectAsState()
    
    when (state) {
        is UserState.Loading -> CircularProgressIndicator()
        is UserState.Success -> UserList(
            users = state.users,
            onDelete = { viewModel.processIntent(UserIntent.DeleteUser(it)) }
        )
        is UserState.Error -> ErrorMessage(state.message)
        is UserState.Idle -> {}
    }
}
```

### Q2: Explain unidirectional data flow in MVI

**Answer:**

**Theoretical Concept:**

Unidirectional data flow is a core principle that ensures data moves in a single, predictable direction through the application. This contrasts with bidirectional data binding where data can flow in multiple directions, making it harder to reason about the application state.

**The Data Flow Cycle:**

Unidirectional data flow means data flows in one direction only, creating a predictable cycle:

```
User Action → Intent → ViewModel → State → View → (loop)
```

**Detailed Flow Explanation:**

1. **User Action**: User interacts with the UI (clicks button, types text, etc.)
2. **Intent Creation**: The view creates an Intent object describing the action
3. **Intent Processing**: ViewModel receives and processes the Intent
4. **State Update**: ViewModel produces a new State based on the Intent
5. **UI Update**: View observes the new State and updates the UI

**Key Properties:**

- **Circular but Unidirectional**: The flow forms a circle but data never goes backwards
- **No Sideways Communication**: Views don't communicate with each other directly
- **Single Authority**: ViewModel is the only place that can change state
- **Predictable**: Every state change comes from an intent, making debugging easier

**Why This Matters:**

In traditional Android development, you might have multiple places updating the same UI, callbacks, observers, and event listeners going in many directions. MVI's unidirectional flow means:
- You can trace exactly why UI changed (look at the intent history)
- You can replay user actions (intent log)
- You can time-travel debug (replay to any point in history)
- State changes are explicit and intentional

**Benefits:**

- Predictability: No surprise state changes
- Debuggability: Clear cause-and-effect chain
- Testability: Easy to test state transitions
- Maintainability: Clear separation of concerns
- Thread-safety: Single authority for state updates

```kotlin
// 1. User action creates Intent
Button(onClick = { 
    viewModel.processIntent(UserIntent.LoadUsers()) 
})

// 2. ViewModel processes Intent
private fun processIntent(intent: UserIntent) {
    when (intent) {
        is UserIntent.LoadUsers -> loadUsers()
        is UserIntent.DeleteUser -> deleteUser(intent.id)
    }
}

// 3. ViewModel updates State
private fun loadUsers() {
    viewModelScope.launch {
        _state.value = UserState.Loading
        val users = repository.getUsers()
        _state.value = UserState.Success(users)
    }
}

// 4. View observes State
val state by viewModel.state.collectAsState()
when (state) {
    is UserState.Success -> displayUsers(state.users)
}
```

**Benefits:**
- Predictable state changes
- Easier debugging
- Thread-safe
- Testable

### Q3: How to handle side effects in MVI?

**Answer:**

Side effects are observable events like navigation, showing toasts:

```kotlin
// Side effects as sealed class
sealed class SideEffect {
    data class ShowToast(val message: String) : SideEffect()
    object NavigateBack : SideEffect()
    data class NavigateTo(val destination: String) : SideEffect()
}

class UserViewModel : ViewModel() {
    
    private val _effects = Channel<SideEffect>(Channel.BUFFERED)
    val effects: Flow<SideEffect> = _effects.receiveAsFlow()
    
    fun deleteUser(userId: String) {
        viewModelScope.launch {
            try {
                repository.deleteUser(userId)
                _effects.send(SideEffect.ShowToast("User deleted"))
                _effects.send(SideEffect.NavigateBack)
            } catch (e: Exception) {
                _effects.send(SideEffect.ShowToast("Error: ${e.message}"))
            }
        }
    }
}

// Handle side effects in View
@Composable
fun UserScreen(viewModel: UserViewModel) {
    val state by viewModel.state.collectAsState()
    
    LaunchedEffect(Unit) {
        viewModel.effects.collect { effect ->
            when (effect) {
                is SideEffect.ShowToast -> showToast(effect.message)
                is SideEffect.NavigateBack -> navController.popBackStack()
                is SideEffect.NavigateTo -> navController.navigate(effect.destination)
            }
        }
    }
}
```

### Q4: What are sealed classes for state management in MVI?

**Answer:**

Sealed classes represent all possible states:

```kotlin
sealed class UserState {
    object Idle : UserState()
    object Loading : UserState()
    data class Success(
        val users: List<User>,
        val searchQuery: String = ""
    ) : UserState()
    data class Error(
        val message: String,
        val retry: () -> Unit
    ) : UserState()
}

// Exhaustive when expression
fun handleState(state: UserState) {
    when (state) {
        is UserState.Idle -> {
            // Handle idle
        }
        is UserState.Loading -> {
            // Show loader
        }
        is UserState.Success -> {
            // Display users
        }
        is UserState.Error -> {
            // Show error and retry button
        }
    }
}
```

### Q5: How to test MVI architecture?

**Answer:**

```kotlin
class UserViewModelTest {
    
    private lateinit var viewModel: UserViewModel
    private lateinit var repository: UserRepository
    
    @Before
    fun setup() {
        repository = mockk()
        viewModel = UserViewModel(repository)
    }
    
    @Test
    fun `process LoadUsers intent updates state`() = runTest {
        // Given
        val users = listOf(User("1", "John"))
        coEvery { repository.getUsers() } returns users
        
        // When
        viewModel.processIntent(UserIntent.LoadUsers)
        
        // Then
        val state = viewModel.state.value
        assertTrue(state is UserState.Success)
        assertEquals(users, (state as UserState.Success).users)
    }
    
    @Test
    fun `process DeleteUser intent triggers deletion`() = runTest {
        // Given
        val userId = "1"
        coEvery { repository.deleteUser(userId) } just Runs
        
        // When
        viewModel.processIntent(UserIntent.DeleteUser(userId))
        
        // Then
        coVerify { repository.deleteUser(userId) }
    }
}
```

### Q6: Compare MVI with MVVM

**Answer:**

| Aspect | MVVM | MVI |
|--------|------|-----|
| State Management | Multiple state properties | Single state object |
| Data Flow | Bidirectional | Unidirectional |
| Intent Pattern | Action methods | Intent objects |
| Side Effects | Callbacks/Events | Separate flow |
| Testing | Multiple properties to verify | Single state to verify |

**MVVM:**
```kotlin
class UserViewModel : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading
    
    fun loadUsers() { }
    fun deleteUser(id: String) { }
}
```

**MVI:**
```kotlin
class UserViewModel : ViewModel() {
    private val _state = MutableStateFlow<UserState>(UserState.Idle)
    val state: StateFlow<UserState> = _state.asStateFlow()
    
    fun processIntent(intent: UserIntent) { }
}
```

### Q7: How is MVI similar to Redux?

**Answer:**

Similarities between MVI and Redux:

**Redux Pattern:**
```javascript
// Actions
const loadUsers = () => ({ type: 'LOAD_USERS' });

// Reducer
function userReducer(state, action) {
    switch (action.type) {
        case 'LOAD_USERS':
            return { ...state, loading: true };
        case 'LOAD_SUCCESS':
            return { ...state, users: action.data, loading: false };
        default:
            return state;
    }
}
```

**MVI Pattern:**
```kotlin
// Intent
sealed class UserIntent {
    object LoadUsers : UserIntent()
}

// Reducer (in ViewModel)
fun reduceIntent(intent: UserIntent, currentState: UserState): UserState {
    return when (intent) {
        is UserIntent.LoadUsers -> UserState.Loading
        // ...
    }
}
```

**Common Patterns:**
- Unidirectional data flow
- Immutable state
- Actions/Intents trigger state changes
- Predictable state updates

### Q8: When to use MVI pattern?

**Answer:**

**Use MVI when:**
- Complex UI with multiple states
- Need predictable state management
- Want exhaustive state handling
- Need better testability
- Working with reactive programming

**When NOT to use:**
- Simple UI with few states
- Small projects
- Team unfamiliar with reactive programming
- Overkill for simple features

**Example: Complex Form**

```kotlin
sealed class FormState {
    data class Editing(
        val name: String,
        val email: String,
        val errors: List<String> = emptyList()
    ) : FormState()
    
    object Submitting : FormState()
    data class Success(val message: String) : FormState()
    data class Error(val message: String) : FormState()
}

// MVI helps manage complex form state
```

---

**Previous:** [Jetpack Compose](jetpack-compose.md)

**Next:** [Android KMM](android-kmm.md)

