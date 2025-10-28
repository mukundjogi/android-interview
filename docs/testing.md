---
layout: default
title: "Testing - Interview Preparation"
---

# Testing

## Table of Contents
- [Unit Tests](#unit-tests)
- [Integration Tests](#integration-tests)
- [UI Tests](#ui-tests)
- [Mocking](#mocking)
- [Interview Questions & Answers](#interview-questions--answers)

## Unit Tests

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
    fun `load users successfully updates state`() = runTest {
        // Given
        val users = listOf(User("1", "John"))
        coEvery { repository.getUsers() } returns users
        
        // When
        viewModel.loadUsers()
        
        // Then
        val state = viewModel.uiState.value
        assertTrue(state is UiState.Success)
    }
}
```

## Unit Tests

**What are Unit Tests?**

Unit tests are like quality checks for individual pieces of your code. Think of them as testing each ingredient in a recipe separately before cooking the whole meal.

**Why Unit Tests Matter:**

- **Fast Feedback**: Know immediately if something breaks
- **Confidence**: Make changes without fear of breaking things
- **Documentation**: Tests show how your code should work
- **Quality**: Catch bugs before users do

**What Makes a Good Unit Test:**

- **Fast**: Run in milliseconds
- **Isolated**: Test one thing at a time
- **Repeatable**: Same result every time
- **Clear**: Easy to understand what it's testing

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
    fun `load users successfully updates state`() = runTest {
        // Given - Set up the test data
        val users = listOf(User("1", "John"))
        coEvery { repository.getUsers() } returns users
        
        // When - Execute the action
        viewModel.loadUsers()
        
        // Then - Verify the result
        val state = viewModel.uiState.value
        assertTrue(state is UiState.Success)
    }
}
```

**How it works:**
1. **Given**: Set up test data and mock responses
2. **When**: Call the method you want to test
3. **Then**: Check that the result is what you expected

## Integration Tests

**What are Integration Tests?**

Integration tests check how different parts of your app work together. It's like testing that all the ingredients work well together in the actual recipe.

**Why Integration Tests Matter:**

- **Real Interactions**: Test actual connections between components
- **Data Flow**: Verify data moves correctly between layers
- **Database**: Test real database operations
- **API**: Test actual network calls (or mocked ones)

**Integration Test Example:**

```kotlin
@RunWith(AndroidJUnit4::class)
class UserRepositoryIntegrationTest {
    @get:Rule
    val instantExecutorRule = InstantTaskExecutorRule()
    
    private lateinit var database: AppDatabase
    private lateinit var repository: UserRepository
    
    @Before
    fun setup() {
        // Create in-memory database for testing
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            AppDatabase::class.java
        ).build()
        repository = UserRepositoryImpl(database.userDao())
    }
    
    @Test
    fun `insert and retrieve user works`() = runTest {
        // Given
        val user = User(1, "John")
        
        // When
        repository.insertUser(user)
        val retrieved = repository.getUser(1)
        
        // Then
        assertEquals(user, retrieved)
    }
}
```

**What this tests:**
- Database operations work correctly
- Data is stored and retrieved properly
- Repository layer functions as expected

## UI Tests

**What are UI Tests?**

UI tests check your app from the user's perspective. They simulate real user interactions like tapping buttons, typing text, and swiping screens.

**Why UI Tests Matter:**

- **User Experience**: Ensure the app works for real users
- **End-to-End**: Test complete user workflows
- **Regression**: Catch when changes break existing features
- **Confidence**: Know the app works before releasing

**UI Test Example:**

```kotlin
@RunWith(AndroidJUnit4::class)
class MainActivityTest {
    
    @get:Rule
    val activityRule = ActivityScenarioRule(MainActivity::class.java)
    
    @Test
    fun login_success_navigates_to_home() {
        // Given - User is on login screen
        
        // When - User enters credentials and taps login
        onView(withId(R.id.email_edit))
            .perform(typeText("test@example.com"))
        
        onView(withId(R.id.password_edit))
            .perform(typeText("password"))
        
        onView(withId(R.id.login_button))
            .perform(click())
        
        // Then - User should see home screen
        onView(withId(R.id.home_screen))
            .check(matches(isDisplayed()))
    }
}
```

**What this tests:**
- Login form works correctly
- Navigation happens after successful login
- UI elements are visible and clickable

## Mocking

**What is Mocking?**

Mocking is creating fake versions of dependencies for testing. It's like using a stunt double in a movie - looks real but isn't the actual actor.

**Why Mock?**

- **Speed**: Avoid slow operations like network calls
- **Control**: Make dependencies behave exactly how you want
- **Isolation**: Test one component without depending on others
- **Reliability**: Tests don't fail because of external factors

**Types of Test Doubles:**

1. **Mocks** - Fake objects that verify interactions
2. **Stubs** - Fake objects that return predetermined values
3. **Fakes** - Working implementations with limited functionality

**Mock Example:**

```kotlin
class DataRepositoryTest {
    @Test
    fun `fetch data calls api`() = runTest {
        // Create a mock API service
        val mockApi = mockk<ApiService>()
        coEvery { mockApi.getData() } returns "test data"
        
        // Use the mock in your test
        val repository = DataRepository(mockApi)
        val result = repository.fetchData()
        
        // Verify the result
        assertEquals("test data", result)
        
        // Verify the API was called
        coVerify { mockApi.getData() }
    }
}
```

**How it works:**
1. Create a mock of the dependency
2. Tell the mock what to return
3. Use the mock in your test
4. Verify the mock was called correctly

## Interview Questions & Answers

### Q1: What are unit, integration, and UI tests?

**Answer:**

**Theoretical Overview:**

The testing pyramid concept, popularized by Mike Cohn, provides a framework for understanding different test types. Each layer serves a specific purpose and has different characteristics regarding speed, cost, and maintainability.

**The Testing Pyramid:**
- **Unit Tests** (base): Fast, cheap, numerous. Test individual components in isolation.
- **Integration Tests** (middle): Medium speed, test component interactions.
- **UI Tests** (top): Slow, expensive, fewer. Test end-to-end user workflows.

**Why This Structure Matters:**
1. **Speed**: Unit tests run in milliseconds, UI tests take seconds/minutes
2. **Cost**: Unit tests are cheap to maintain, UI tests are expensive
3. **Feedback Loop**: Fast tests mean quick feedback on code changes
4. **Reliability**: Unit tests are more stable, UI tests flaky due to multiple dependencies

**Unit Tests:**
```kotlin
// Test individual function/class
class CalculatorTest {
    @Test
    fun `addition works correctly`() {
        val calculator = Calculator()
        val result = calculator.add(2, 3)
        assertEquals(5, result)
    }
}
```
- Fast execution (milliseconds)
- Isolated testing
- Mock dependencies

**Integration Tests:**
```kotlin
// Test component interactions
@RunWith(AndroidJUnit4::class)
class UserRepositoryIntegrationTest {
    @get:Rule
    val instantExecutorRule = InstantTaskExecutorRule()
    
    private lateinit var database: AppDatabase
    private lateinit var repository: UserRepository
    
    @Before
    fun setup() {
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            AppDatabase::class.java
        ).build()
        repository = UserRepositoryImpl(database.userDao())
    }
    
    @Test
    fun `insert and retrieve user works`() = runTest {
        val user = User(1, "John")
        repository.insertUser(user)
        
        val retrieved = repository.getUser(1)
        assertEquals(user, retrieved)
    }
}
```

**UI Tests (Espresso):**
```kotlin
@RunWith(AndroidJUnit4::class)
class MainActivityTest {
    
    @get:Rule
    val activityRule = ActivityScenarioRule(MainActivity::class.java)
    
    @Test
    fun login_success_navigates_to_home() {
        onView(withId(R.id.email_edit))
            .perform(typeText("test@example.com"))
        
        onView(withId(R.id.password_edit))
            .perform(typeText("password"))
        
        onView(withId(R.id.login_button))
            .perform(click())
        
        onView(withId(R.id.home_screen))
            .check(matches(isDisplayed()))
    }
}
```

### Q2: How to test ViewModels?

**Answer:**

**Using Coroutine Test:**
```kotlin
class UserViewModelTest {
    
    private lateinit var viewModel: UserViewModel
    private lateinit var repository: UserRepository
    
    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()
    
    @Before
    fun setup() {
        repository = mockk()
        viewModel = UserViewModel(repository)
    }
    
    @Test
    fun `load users updates state correctly`() = runTest {
        // Given
        val users = listOf(User("1", "John"))
        coEvery { repository.getUsers() } returns flowOf(users)
        
        // When
        viewModel.loadUsers()
        
        // Then
        val state = viewModel.uiState.value
        assertTrue(state is UiState.Success)
        assertEquals(users, (state as UiState.Success).data)
    }
    
    @Test
    fun `error is handled correctly`() = runTest {
        // Given
        coEvery { repository.getUsers() } throws NetworkException("Error")
        
        // When
        viewModel.loadUsers()
        
        // Then
        val state = viewModel.uiState.value
        assertTrue(state is UiState.Error)
    }
}
```

### Q3: What are test doubles (mocks, stubs, fakes)?

**Answer:**

**1. Mocks (MockK):**
```kotlin
// Mock external dependencies
interface ApiService {
    suspend fun getData(): String
}

class DataRepository(private val api: ApiService) {
    suspend fun fetchData() = api.getData()
}

class DataRepositoryTest {
    @Test
    fun `fetch data calls api`() = runTest {
        val mockApi = mockk<ApiService>()
        coEvery { mockApi.getData() } returns "test data"
        
        val repository = DataRepository(mockApi)
        val result = repository.fetchData()
        
        assertEquals("test data", result)
        coVerify { mockApi.getData() }
    }
}
```

**2. Stubs:**
```kotlin
// Provide canned responses
class StubApiService : ApiService {
    override suspend fun getData() = "stub data"
}
```

**3. Fakes:**
```kotlin
// Working implementation with limited capability
class FakeUserDatabase : UserDataSource {
    private val users = mutableListOf<User>()
    
    override suspend fun getAllUsers(): List<User> = users
    
    override suspend fun insertUser(user: User) {
        users.add(user)
    }
}
```

### Q4: How to use MockK for mocking?

**Answer:**

**Conceptual Understanding of Mocking:**

Mocking is a fundamental testing technique where you replace real dependencies with controllable fake implementations. This serves several purposes:
1. **Isolation**: Test the component in isolation without external dependencies
2. **Speed**: Avoid slow operations like network calls, database I/O
3. **Controllability**: Control the behavior of dependencies to test edge cases
4. **Determinism**: Make tests repeatable by controlling external factors

**MockK Philosophy:**

MockK is a mocking library for Kotlin that uses three main strategies:
- **Mocking**: Replace an object with a controllable version
- **Stubbing**: Define return values for method calls
- **Verification**: Check that methods were called with expected parameters

**Basic Mocking:**
```kotlin
// Mock object
val mockRepository = mockk<UserRepository>()

// Specify behavior
every { mockRepository.getUser(any()) } returns User("1", "John")
coEvery { mockRepository.getUserAsync(any()) } returns User("2", "Jane")

// Use in test
val viewModel = UserViewModel(mockRepository)
```

**Verification:**
```kotlin
@Test
fun `verify method calls`() {
    val mockApi = mockk<ApiService>()
    every { mockApi.fetchData() } returns "data"
    
    val service = MyService(mockApi)
    service.processData()
    
    verify(exactly = 1) { mockApi.fetchData() }
}
```

**Slot for capturing arguments:**
```kotlin
@Test
fun `capture arguments`() {
    val mockApi = mockk<ApiService>()
    val slot = slot<String>()
    every { mockApi.saveData(capture(slot)) } just Runs
    
    val service = MyService(mockApi)
    service.save("test")
    
    assertEquals("test", slot.captured)
}
```

### Q5: How to write Espresso tests?

**Answer:**

```kotlin
@RunWith(AndroidJUnit4::class)
class UserFlowTest {
    
    @get:Rule
    val activityRule = ActivityScenarioRule(MainActivity::class.java)
    
    @Test
    fun complete_user_journey() {
        // 1. Login
        onView(withId(R.id.email_field))
            .perform(typeText("user@example.com"))
        
        onView(withId(R.id.password_field))
            .perform(typeText("password"))
        
        onView(withId(R.id.login_button))
            .perform(click())
        
        // 2. Wait for home screen
        onView(withId(R.id.home_screen))
            .check(matches(isDisplayed()))
        
        // 3. Navigate to profile
        onView(withId(R.id.profile_button))
            .perform(click())
        
        // 4. Verify profile screen
        onView(withId(R.id.profile_screen))
            .check(matches(isDisplayed()))
        
        // 5. Check user data
        onView(withId(R.id.email_text))
            .check(matches(withText("user@example.com")))
    }
    
    @Test
    fun scroll_and_click_item() {
        onView(withId(R.id.recycler_view))
            .perform(actionOnItem(
                withText("Item"),
                click()
            ))
        
        onView(withId(R.id.detail_screen))
            .check(matches(isDisplayed()))
    }
}
```

### Q6: How to test coroutines in tests?

**Answer:**

**Conceptual Background:**

Testing coroutines is challenging because they involve asynchronous execution, time delays, and concurrent operations. The key challenges are:
1. **Timing**: Real delays would make tests slow and flaky
2. **Determinism**: Need predictable execution order
3. **Cancellation**: Need to verify cancellation behavior
4. **Thread Safety**: Need to ensure thread-safe state changes

**Testing Strategy:**

The solution is `TestCoroutineDispatcher` which:
- Runs all coroutines synchronously in tests
- Provides control over virtual time
- Makes async code testable like sync code
- Ensures deterministic test execution

**Test with TestCoroutineDispatcher:**
```kotlin
class CoroutineViewModelTest {
    
    @get:Rule
    val mainCoroutineRule = MainCoroutineRule()
    
    @Before
    fun setup() {
        Dispatchers.setMain(Dispatchers.Unconfined)
    }
    
    @After
    fun teardown() {
        Dispatchers.resetMain()
    }
    
    @Test
    fun `test coroutine in viewmodel`() = runTest {
        val repository = mockk<UserRepository>()
        coEvery { repository.getUsers() } returns flowOf(listOf(User("1", "John")))
        
        val viewModel = UserViewModel(repository)
        viewModel.loadUsers()
        
        advanceUntilIdle()
        
        val state = viewModel.uiState.value
        assertTrue(state is UiState.Success)
    }
}

class MainCoroutineRule : TestWatcher() {
    val testDispatcher = TestCoroutineDispatcher()
    
    override fun starting(description: Description) {
        Dispatchers.setMain(testDispatcher)
    }
    
    override fun finished(description: Description) {
        Dispatchers.resetMain()
        testDispatcher.cleanupTestCoroutines()
    }
}
```

### Q7: Explain test-driven development (TDD)

**Answer:**

TDD follows Red-Green-Refactor cycle:

**Red: Write failing test**
```kotlin
@Test
fun `email validator rejects invalid email`() {
    val validator = EmailValidator()
    assertFalse(validator.isValid("invalid"))
}
```

**Green: Write minimum code to pass**
```kotlin
class EmailValidator {
    fun isValid(email: String): Boolean {
        return email.contains("@")
    }
}
```

**Refactor: Improve code while keeping tests green**
```kotlin
class EmailValidator {
    fun isValid(email: String): Boolean {
        val regex = Regex("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$")
        return regex.matches(email)
    }
}
```

**Complete TDD Example:**
```kotlin
// 1. Test first
class UserManagerTest {
    @Test
    fun `creates user with valid data`() {
        val manager = UserManager()
        val user = manager.createUser("John", "john@example.com")
        
        assertNotNull(user)
        assertEquals("John", user.name)
    }
}

// 2. Implement
class UserManager {
    fun createUser(name: String, email: String): User {
        return User(name, email)
    }
}

// 3. Add more tests
@Test
fun `throws error for invalid email`() {
    val manager = UserManager()
    assertThrows<ValidationException> {
        manager.createUser("John", "invalid")
    }
}
```

---

**Previous:** [Memory Management](memory-management.md)

**Next:** [Gradle & Maven](gradle-maven.md)

