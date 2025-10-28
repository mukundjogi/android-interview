---
layout: default
title: "Jetpack Compose - Interview Preparation"
---

# Jetpack Compose

## Table of Contents
- [Introduction](#introduction)
- [Composable Functions](#composable-functions)
- [State Management](#state-management)
- [Recomposition](#recomposition)
- [Navigation](#navigation)
- [Interview Questions & Answers](#interview-questions--answers)

## State Management

**What is State in Compose?**

State in Jetpack Compose is any data that can change over time and affects what the UI displays. Think of state as the "memory" of your UI components - it remembers what the user has done and what the app should show.

**Why State Management Matters:**

- **Reactivity**: When state changes, Compose automatically updates the UI
- **Persistence**: State survives recomposition (when Compose rebuilds parts of your UI)
- **Sharing**: State can be shared between different composable functions

**Key Concepts:**

1. **State**: Data that can change (like a counter value)
2. **Remember**: Keeps state alive during recomposition
3. **MutableStateOf**: Creates state that can be modified
4. **Recomposition**: When Compose rebuilds UI due to state changes

```kotlin
@Composable
fun Counter() {
    // This creates a state variable that remembers its value
    // even when the composable is recomposed
    var count by remember { mutableStateOf(0) }
    
    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}
```

**How it works:**
1. `remember` ensures the state survives recomposition
2. `mutableStateOf(0)` creates state with initial value 0
3. When button is clicked, `count++` changes the state
4. Compose sees the state change and recomposes the UI
5. The Text shows the new count value

## Recomposition

**What is Recomposition?**

Recomposition is Compose's way of updating the UI when something changes. Think of it like this: when you change the state (like clicking a button), Compose looks at what parts of the UI depend on that state and rebuilds only those parts.

**Why is Recomposition Important?**

- **Efficiency**: Only rebuilds what actually changed, not the entire screen
- **Automatic**: You don't need to manually tell Compose what to update
- **Smart**: Compose tracks dependencies automatically

**How Recomposition Works:**

1. **State Change**: Something changes (user clicks button, data loads, etc.)
2. **Dependency Tracking**: Compose knows which composables read that state
3. **Selective Rebuild**: Only those composables are recomposed
4. **UI Update**: The screen shows the new state

**Key Concepts:**

- **Smart Recomputation**: Compose uses a "snapshot" system to track state reads
- **Efficient Diffing**: Only changed parts re-render, similar to how React works
- **Stable Parameters**: Compose optimizes by identifying which parameters don't change
- **Intelligent Skipping**: If parameters haven't changed, Compose skips recomposition

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    
    // Only this composable recomposes when count changes
    Column {
        Button(onClick = { count++ }) {
            Text("Count: $count")
        }
        // This won't recompose if count changes
        ExpensiveComponent()
    }
}

@Composable
fun ExpensiveComponent() {
    // Heavy computation
    Text("Static content")
}
```

**What happens here:**
1. When `count` changes, only the `Column` and its children recompose
2. `ExpensiveComponent` doesn't recompose because it doesn't read `count`
3. This makes the app faster and more efficient

## Navigation

**What is Navigation in Compose?**

Navigation in Compose is how you move between different screens (composables) in your app. It's like having a map that tells your app how to get from one screen to another.

**Key Concepts:**

- **NavController**: The "driver" that handles navigation
- **NavHost**: The "container" that holds all your screens
- **Routes**: The "addresses" of your screens (like "home", "profile")
- **Arguments**: Data you pass between screens

**Why Navigation Matters:**

- **User Experience**: Smooth transitions between screens
- **State Management**: Properly handles screen state
- **Deep Linking**: Allows users to jump directly to specific screens
- **Back Stack**: Manages the history of screens

```kotlin
@Composable
fun MainNavigation() {
    val navController = rememberNavController()
    
    NavHost(
        navController = navController,
        startDestination = "home" // Where to start
    ) {
        composable("home") { HomeScreen(navController) }
        composable("profile/{userId}") { backStackEntry ->
            ProfileScreen(
                userId = backStackEntry.arguments?.getString("userId") ?: ""
            )
        }
        composable("settings") { SettingsScreen() }
    }
}
```

**How it works:**
1. `rememberNavController()` creates a navigation controller
2. `NavHost` defines all possible screens and their routes
3. `composable("home")` defines a screen with route "home"
4. `composable("profile/{userId}")` defines a screen that accepts a userId parameter
5. You can navigate by calling `navController.navigate("route")`

## Interview Questions & Answers

### Q1: What is Jetpack Compose and how does it differ from XML layouts?

**Answer:**

Jetpack Compose is Android's modern declarative UI toolkit for building native Android interfaces using Kotlin. It revolutionizes Android UI development by moving from the traditional imperative XML-based approach to a declarative, reactive paradigm.

**Theoretical Concepts:**

**Declarative Programming:**
In Compose, you describe the UI based on state rather than manipulating the view hierarchy. You declare what the UI should look like given a particular state, and Compose handles the rest automatically.

**Imperative vs Declarative Approach:**
- **Imperative (XML/Views)**: You explicitly manipulate views, call methods like setText(), setVisibility(), etc. The developer is responsible for transitioning between states.
- **Declarative (Compose)**: You describe what you want, Compose figures out how to achieve it. The framework handles the view updates.

**Key Conceptual Differences:**
1. **State-driven UI**: The UI is a function of state. State changes trigger automatic UI updates.
2. **Composition over Inheritance**: Reusable composable functions instead of view class hierarchies.
3. **Unidirectional Data Flow**: Data flows down (from parent to child), events flow up.
4. **Recomposition**: Only changed parts re-render, not the entire view tree.

**Key Differences:**
- Declarative vs Imperative: Describe UI vs manipulate views
- Kotlin vs XML: Code-based vs markup language
- Recomposition vs Mutation: Automatic diffing vs manual updates
- State-driven UI: Reactive updates vs event-driven updates

```kotlin
// Compose: Declarative
@Composable
fun UserProfile(user: User) {
    Column {
        Text(user.name)
        Text(user.email)
    }
}

// XML: Imperative - need to manually update views
class UserActivity : AppCompatActivity() {
    private lateinit var nameText: TextView
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user)
        nameText = findViewById(R.id.name_text)
    }
    
    fun updateName(name: String) {
        nameText.text = name // Manual update
    }
}
```

### Q2: Explain recomposition in Jetpack Compose

**Answer:**

**Theoretical Background:**

Recomposition is the process by which Compose re-invokes composable functions to reflect state changes. Unlike traditional Android views where you manually update each view, Compose automatically tracks state dependencies and only recomposes what actually changed.

**Key Principles:**

1. **Smart Recomputation**: Compose uses a "snapshot" system to track state reads. When a state value changes, Compose marks all composables that read that state as invalidated.

2. **Efficient Diffing**: Compose uses intelligent algorithms to determine what actually changed, similar to how React works. Only the minimal set of composables that need updating are recomposed.

3. **Stable Parameters**: Compose optimizes by identifying which parameters are stable (don't change) and skipping recomposition of composables that only depend on stable data.

4. **Intelligent Skipping**: If a composable's parameters haven't changed (using structural equality), Compose may completely skip recomposition of that composable.

**How It Works:**

The recomposition process involves three stages:
1. **Tracking**: State reads are tracked during composition
2. **Invalidation**: When state changes, affected composables are marked as invalid
3. **Recomposition**: Invalid composables are re-executed

Recomposition is when Compose re-executes composables that might have changed.

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    
    // Only this composable recomposes when count changes
    Column {
        Button(onClick = { count++ }) {
            Text("Count: $count")
        }
        // This won't recompose if count changes
        ExpensiveComponent()
    }
}

@Composable
fun ExpensiveComponent() {
    // Heavy computation
    Text("Static content")
}
```

**Optimization:**
```kotlin
@Composable
fun UserProfile(user: User) {
    // Only recomposes when specific user property changes
    Column {
        Text(user.name) // Only recomposes if name changes
        Text(user.email) // Only recomposes if email changes
    }
}
```

### Q3: Explain state management in Compose (@State, @StateObject)

**Answer:**

**@State:**
```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    
    // State is local to this composable
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}
```

**@StateObject for ViewModel:**
```kotlin
@Composable
fun UserScreen(viewModel: UserViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsState()
    
    when (uiState) {
        is UiState.Loading -> CircularProgressIndicator()
        is UiState.Success -> UserList(uiState.data)
        is UiState.Error -> ErrorMessage(uiState.message)
    }
}

class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow<UiState<List<User>>>(UiState.Idle)
    val uiState: StateFlow<UiState<List<User>>> = _uiState.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            try {
                val users = repository.getUsers()
                _uiState.value = UiState.Success(users)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message)
            }
        }
    }
}
```

### Q4: How to handle navigation in Jetpack Compose?

**Answer:**

**Setup:**
```kotlin
@Composable
fun MainNavigation() {
    val navController = rememberNavController()
    
    NavHost(
        navController = navController,
        startDestination = "home"
    ) {
        composable("home") { HomeScreen(navController) }
        composable("profile/{userId}") { backStackEntry ->
            ProfileScreen(
                userId = backStackEntry.arguments?.getString("userId") ?: ""
            )
        }
        composable("settings") { SettingsScreen() }
    }
}
```

**Navigate:**
```kotlin
@Composable
fun HomeScreen(navController: NavController) {
    Column {
        Button(onClick = { 
            navController.navigate("profile/user123") 
        }) {
            Text("Go to Profile")
        }
    }
}
```

### Q5: What is state hoisting in Compose?

**Answer:**

State hoisting moves state up to make composables stateless and reusable.

```kotlin
// BAD: State in child
@Composable
fun CounterButton() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}

// GOOD: State hoisted to parent
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    CounterButton(
        count = count,
        onCountChange = { count = it }
    )
}

@Composable
fun CounterButton(
    count: Int,
    onCountChange: (Int) -> Unit
) {
    Button(onClick = { onCountChange(count + 1) }) {
        Text("Count: $count")
    }
}
```

### Q6: How to optimize Compose performance?

**Answer:**

**1. Avoid unnecessary recomposition:**
```kotlin
// BAD
@Composable
fun ItemList(items: List<Item>) {
    LazyColumn {
        items(items) { item ->
            ItemCard(item) // Recreates on every recomposition
        }
    }
}

// GOOD
@Composable
fun ItemList(items: List<Item>) {
    LazyColumn {
        items(items, key = { it.id }) { item ->
            ItemCard(item) // Key ensures proper recycling
        }
    }
}
```

**2. Use derivedStateOf:**
```kotlin
@Composable
fun FilteredList(items: List<Item>, searchQuery: String) {
    val filteredItems by remember {
        derivedStateOf {
            items.filter { it.name.contains(searchQuery) }
        }
    }
    LazyColumn {
        items(filteredItems) { item -> ItemCard(item) }
    }
}
```

**3. Use compositionLocalProvider wisely:**
```kotlin
@Composable
fun Application() {
    val viewModel = hiltViewModel()
    CompositionLocalProvider(LocalViewModel provides viewModel) {
        MyContent()
    }
}
```

### Q7: Explain Compose lifecycle and side effects

**Answer:**

**Lifecycle:**
```kotlin
@Composable
fun MyComponent() {
    DisposableEffect(Unit) {
        println("Composed")
        onDispose {
            println("Disposed")
        }
    }
}
```

**Side Effects:**
```kotlin
@Composable
fun UserProfile(userId: String) {
    val viewModel: UserViewModel = hiltViewModel()
    
    LaunchedEffect(userId) {
        // Execute when userId changes
        viewModel.loadUser(userId)
    }
    
    val user by viewModel.user.collectAsState()
    
    // UI
    Column {
        Text(user?.name ?: "Loading...")
    }
}
```

### Q8: How to test Compose UI?

**Answer:**

```kotlin
class ComposeTest {
    
    @get:Rule
    val composeTestRule = createComposeRule()
    
    @Test
    fun counter_increments_when_button_clicked() {
        composeTestRule.setContent {
            Counter()
        }
        
        // Verify initial state
        composeTestRule.onNodeWithText("Count: 0").assertExists()
        
        // Perform action
        composeTestRule.onNodeWithText("Increment").performClick()
        
        // Verify updated state
        composeTestRule.onNodeWithText("Count: 1").assertExists()
    }
    
    @Test
    fun list_displays_items() {
        composeTestRule.setContent {
            ItemList(items = listOf(
                Item("1", "Item 1"),
                Item("2", "Item 2")
            ))
        }
        
        composeTestRule.onNodeWithText("Item 1").assertExists()
        composeTestRule.onNodeWithText("Item 2").assertExists()
    }
}
```

### Q9: Compare LazyColumn with RecyclerView

**Answer:**

**LazyColumn:**
```kotlin
@Composable
fun ItemList(items: List<Item>) {
    LazyColumn {
        items(items) { item ->
            ItemCard(item)
        }
    }
}
```

**RecyclerView:**
```kotlin
class ItemAdapter(private val items: List<Item>) : RecyclerView.Adapter<ItemViewHolder>() {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ItemViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_card, parent, false)
        return ItemViewHolder(view)
    }
    
    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) {
        holder.bind(items[position])
    }
    
    override fun getItemCount() = items.size
}
```

**Advantages of LazyColumn:**
- Declarative syntax
- No ViewHolder pattern needed
- Built-in state management
- Composable functions as items

### Q10: How to handle animations in Compose?

**Answer:**

```kotlin
@Composable
fun AnimatedCounter() {
    var count by remember { mutableStateOf(0) }
    
    // Animate number change
    val animatedCount by animateAsInt(targetValue = count)
    
    Button(onClick = { count++ }) {
        Text("Count: $animatedCount")
    }
}

@Composable
fun FadeInOut(visible: Boolean, content: @Composable () -> Unit) {
    val alpha by animateFloatAsState(
        targetValue = if (visible) 1f else 0f
    )
    
    Box(modifier = Modifier.alpha(alpha)) {
        content()
    }
}

@Composable
fun SlideContent(visible: Boolean) {
    AnimatedVisibility(
        visible = visible,
        enter = slideInVertically() + fadeIn(),
        exit = slideOutVertically() + fadeOut()
    ) {
        Text("Animated content")
    }
}
```

---

**Previous:** [Android Architecture](android-architecture.md)

**Next:** [MVI Pattern](mvi-pattern.md)

