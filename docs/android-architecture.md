---
layout: default
title: "Android Architecture - Interview Preparation"
---

# Android Architecture

## Table of Contents
- [MVVM Pattern](#mvvm-pattern)
- [MVI Pattern](#mvi-pattern)
- [Clean Architecture](#clean-architecture)
- [Repository Pattern](#repository-pattern)
- [SOLID Principles](#solid-principles)
- [Interview Questions & Answers](#interview-questions--answers)

## MVVM Pattern

```kotlin
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow<UiState>(UiState.Idle)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
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

## MVVM Pattern

**What is MVVM (Model-View-ViewModel)?**

MVVM is an architectural pattern that separates your app into three main parts. Think of it like organizing a restaurant: the kitchen (Model), the dining room (View), and the waiter (ViewModel) who connects them.

**Why Use MVVM?**

- **Separation of Concerns**: Each part has a specific job
- **Testability**: Easy to test each part separately
- **Maintainability**: Changes in one part don't affect others
- **Reusability**: ViewModels can be shared between different Views

**The Three Components:**

1. **Model** - The "Kitchen" (Data & Business Logic)
   - Contains your data and business rules
   - Handles data operations (API calls, database)
   - Doesn't know about UI

2. **View** - The "Dining Room" (UI)
   - Shows data to the user
   - Handles user interactions
   - Only displays what ViewModel tells it to

3. **ViewModel** - The "Waiter" (Connection Layer)
   - Prepares data for the View
   - Handles user interactions
   - Survives configuration changes
   - Never references View directly

**How MVVM Works:**

```kotlin
// Model - Data and business logic
data class User(val id: Int, val name: String)

interface UserRepository {
    suspend fun getUsers(): List<User>
}

// ViewModel - Prepares data for View
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _users.value = repository.getUsers()
        }
    }
}

// View - Displays data and handles interactions
class UserActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            viewModel.users.collect { users ->
                // Update UI
            }
        }
    }
}
```

**Data Flow:**
1. View calls ViewModel method
2. ViewModel calls Repository
3. Repository returns data
4. ViewModel updates state
5. View observes state and updates UI

## MVI Pattern

**What is MVI (Model-View-Intent)?**

MVI is like MVVM but with stricter rules. Think of it as a more organized restaurant where every order (Intent) goes through a specific process to become food (State).

**Why Use MVI?**

- **Predictable**: Always know where data comes from
- **Debuggable**: Can trace exactly why UI changed
- **Testable**: Easy to test state transitions
- **Maintainable**: Clear separation of concerns

**The Three Components:**

1. **Model** - The "Kitchen" (Business Logic)
   - Processes intents and produces new states
   - Contains all business rules
   - Doesn't know about UI

2. **View** - The "Dining Room" (UI)
   - Shows current state to user
   - Sends intents when user interacts
   - Only displays what Model tells it to

3. **Intent** - The "Order" (User Actions)
   - Represents what user wants to do
   - Like clicking button, typing text
   - Sent from View to Model

**How MVI Works:**

```kotlin
// Intent - User actions
sealed class UserIntent {
    object LoadUsers : UserIntent()
    data class DeleteUser(val userId: String) : UserIntent()
}

// State - UI representation
sealed class UserState {
    object Idle : UserState()
    object Loading : UserState()
    data class Success(val users: List<User>) : UserState()
    data class Error(val message: String) : UserState()
}

// ViewModel - Processes intents
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    
    private val _state = MutableStateFlow<UserState>(UserState.Idle)
    val state: StateFlow<UserState> = _state.asStateFlow()
    
    fun processIntent(intent: UserIntent) {
        when (intent) {
            is UserIntent.LoadUsers -> loadUsers()
            is UserIntent.DeleteUser -> deleteUser(intent.userId)
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
```

**Data Flow:**
1. User action creates Intent
2. ViewModel processes Intent
3. ViewModel updates State
4. View observes State and updates UI

## Clean Architecture

**What is Clean Architecture?**

Clean Architecture is like organizing a city with different districts. Each district has its own purpose, and there are clear rules about how they can interact with each other.

**Why Use Clean Architecture?**

- **Independence**: Each layer can be developed independently
- **Testability**: Easy to test each layer separately
- **Flexibility**: Can change one layer without affecting others
- **Maintainability**: Clear structure makes code easier to maintain

**The Layers:**

1. **Domain Layer** - The "Business District"
   - Contains business logic and rules
   - Independent of frameworks
   - Pure Kotlin/Java code

2. **Data Layer** - The "Storage District"
   - Handles data sources (API, Database)
   - Implements domain interfaces
   - Manages data persistence

3. **Presentation Layer** - The "UI District"
   - Contains UI components
   - Handles user interactions
   - Depends on domain layer

**Clean Architecture Example:**

```kotlin
// Domain Layer - Business Logic
data class User(val id: Int, val name: String)

interface UserRepository {
    suspend fun getUsers(): List<User>
}

class GetUsersUseCase(private val repository: UserRepository) {
    suspend operator fun invoke(): List<User> {
        return repository.getUsers()
    }
}

// Data Layer - Data Sources
class UserRepositoryImpl(
    private val api: UserApi,
    private val dao: UserDao
) : UserRepository {
    
    override suspend fun getUsers(): List<User> {
        val cached = dao.getAllUsers()
        if (cached.isNotEmpty()) return cached
        
        val remote = api.getUsers()
        dao.insertUsers(remote)
        return remote
    }
}

// Presentation Layer - UI
class UserViewModel @Inject constructor(
    private val getUsersUseCase: GetUsersUseCase
) : ViewModel() {
    
    private val _state = MutableStateFlow<UiState>(UiState.Idle)
    val state: StateFlow<UiState> = _state.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _state.value = UiState.Loading
            try {
                val users = getUsersUseCase()
                _state.value = UiState.Success(users)
            } catch (e: Exception) {
                _state.value = UiState.Error(e.message)
            }
        }
    }
}
```

**Dependency Rule:**
- Dependencies point inward
- Domain layer has no dependencies
- Data layer depends on domain
- Presentation layer depends on domain

## Repository Pattern

**What is the Repository Pattern?**

The Repository pattern is like having a personal assistant who knows where to find any information you need. You ask for data, and the assistant figures out whether to get it from the database, API, or cache.

**Why Use Repository Pattern?**

- **Abstraction**: Hide data source complexity
- **Flexibility**: Can change data sources easily
- **Testing**: Easy to mock data sources
- **Caching**: Centralized caching logic

**How Repository Pattern Works:**

```kotlin
// Interface - What the repository can do
interface UserRepository {
    suspend fun getUsers(): Flow<List<User>>
    suspend fun getUserById(id: Int): User?
    suspend fun refresh()
}

// Implementation - How the repository works
class UserRepositoryImpl(
    private val api: UserApi,
    private val dao: UserDao,
    private val networkConnection: NetworkConnection
) : UserRepository {
    
    override suspend fun getUsers(): Flow<List<User>> {
        return flow {
            // Emit cached data first
            emit(dao.getAllUsers())
            
            // Try to fetch from network
            if (networkConnection.isAvailable()) {
                try {
                    val users = api.getUsers()
                    dao.insertUsers(users)
                } catch (e: Exception) {
                    // Return cached data on error
                }
            }
            
            // Emit updated cached data
            emit(dao.getAllUsers())
        }.flowOn(Dispatchers.IO)
    }
    
    override suspend fun getUserById(id: Int): User? {
        val cached = dao.getUserById(id)
        if (cached != null) return cached
        
        return api.getUserById(id)?.also { user ->
            dao.insertUser(user)
        }
    }
    
    override suspend fun refresh() {
        val users = api.getUsers()
        dao.clearAndInsert(users)
    }
}
```

**Benefits:**
- **Single Source of Truth**: One place for all data operations
- **Offline Support**: Can work with cached data
- **Error Handling**: Graceful fallback to cached data
- **Performance**: Fast access to cached data

## SOLID Principles

**What are SOLID Principles?**

SOLID principles are like rules for building good software. Think of them as guidelines that help you write code that's easy to understand, maintain, and extend.

**Why SOLID Principles Matter:**

- **Maintainability**: Code is easier to modify
- **Testability**: Easy to test individual components
- **Flexibility**: Easy to add new features
- **Reusability**: Components can be reused

**The Five Principles:**

### S - Single Responsibility Principle

**What it means:** Each class should have only one reason to change.

```kotlin
// BAD: One class doing multiple things
class BadUserManager {
    fun validateEmail(email: String) { }
    fun saveUser(user: User) { }
    fun sendEmail(user: User) { }
    fun displayUser(user: User) { }
}

// GOOD: Separate responsibilities
class EmailValidator {
    fun validate(email: String): Boolean { }
}

class UserRepository {
    fun saveUser(user: User) { }
}

class EmailService {
    fun sendEmail(user: User) { }
}

class UserViewModel {
    fun displayUser(user: User) { }
}
```

### O - Open/Closed Principle

**What it means:** Open for extension, closed for modification.

```kotlin
// BAD: Modify existing class for new feature
class PaymentProcessor {
    fun processPayment(amount: Double, type: String) {
        when (type) {
            "credit" -> { }
            "debit" -> { }
            "paypal" -> { } // Had to modify
        }
    }
}

// GOOD: Open for extension, closed for modification
interface PaymentMethod {
    fun process(amount: Double)
}

class CreditCardPayment : PaymentMethod {
    override fun process(amount: Double) { }
}

class DebitCardPayment : PaymentMethod {
    override fun process(amount: Double) { }
}

class PaypalPayment : PaymentMethod {
    override fun process(amount: Double) { }
}
```

### L - Liskov Substitution Principle

**What it means:** Subclasses should be substitutable for their base classes.

```kotlin
// BAD: Subclass violates parent behavior
open class View {
    open fun draw() { }
}

class Button : View() {
    override fun draw() {
        throw NotImplementedError() // Violates LSP
    }
}

// GOOD: Subclass maintains parent behavior
class Button : View() {
    override fun draw() {
        // Proper implementation
    }
}
```

### I - Interface Segregation Principle

**What it means:** Clients shouldn't depend on interfaces they don't use.

```kotlin
// BAD: Fat interface
interface DataSource {
    fun read()
    fun write()
    fun delete()
    fun upload() // Some classes don't need this
}

// GOOD: Segregated interfaces
interface Readable {
    fun read()
}

interface Writable {
    fun write()
}

interface Deletable {
    fun delete()
}

interface Uploadable {
    fun upload()
}

class CacheDataSource : Readable, Writable { }
class CloudDataSource : Readable, Writable, Uploadable { }
```

### D - Dependency Inversion Principle

**What it means:** Depend on abstractions, not concretions.

```kotlin
// BAD: Depends on concrete class
class DataService {
    private val database = SqliteDatabase()
    
    fun getData() {
        database.query()
    }
}

// GOOD: Depends on abstraction
class DataService(private val database: Database) {
    fun getData() {
        database.query()
    }
}
```

## Dependency Injection

**What is Dependency Injection?**

Dependency Injection is like having a delivery service that brings you exactly what you need when you need it. Instead of going to get dependencies yourself, they're provided to you.

**Why Use Dependency Injection?**

- **Loose Coupling**: Classes don't create their own dependencies
- **Testability**: Easy to replace dependencies with mocks
- **Flexibility**: Can change implementations easily
- **Maintainability**: Clear dependency relationships

**How Dependency Injection Works:**

```kotlin
// Without DI (Tight coupling)
class OrderViewModel {
    private val api = OrderApi()
    private val db = OrderDatabase()
    
    fun processOrder() {
        val orders = api.getOrders()
        db.saveOrders(orders)
    }
}

// With DI (Loose coupling)
class OrderViewModel(
    private val api: OrderApi,
    private val db: OrderDatabase
) {
    fun processOrder() {
        val orders = api.getOrders()
        db.saveOrders(orders)
    }
}

// Easy to test
class OrderViewModelTest {
    @Test
    fun testProcessOrder() {
        val mockApi = mockk<OrderApi>()
        val mockDb = mockk<OrderDatabase>()
        val viewModel = OrderViewModel(mockApi, mockDb)
        // Test viewModel
    }
}
```

**Benefits:**
- **Testability**: Easy to inject mocks for testing
- **Flexibility**: Can change implementations without changing code
- **Maintainability**: Clear dependency relationships
- **Reusability**: Components can be reused in different contexts

## Interview Questions & Answers

### Q1: Explain MVVM architecture pattern

**Answer:**

MVVM separates business logic from UI:

**Model**: Data and business logic
**View**: UI components (Activity, Fragment, Composable)
**ViewModel**: Prepares and manages data for View

```kotlin
// Model
data class User(val id: Int, val name: String)

// Repository
interface UserRepository {
    suspend fun getUsers(): List<User>
}

// ViewModel
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _users.value = repository.getUsers()
        }
    }
}

// View
class UserActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            viewModel.users.collect { users ->
                // Update UI
            }
        }
    }
}
```

### Q2: What is the difference between MVVM and MVP patterns?

**Answer:**

**MVP (Model-View-Presenter):**

```kotlin
// Model
data class User(val name: String)

// View (Interface)
interface UserView {
    fun showUsers(users: List<User>)
    fun showError(message: String)
}

// Presenter
class UserPresenter(
    private val view: UserView,
    private val repository: UserRepository
) {
    fun loadUsers() {
        repository.getUsers { users, error ->
            if (error != null) {
                view.showError(error.message)
            } else {
                view.showUsers(users)
            }
        }
    }
}
```

**MVVM:**

```kotlin
// ViewModel
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _state = MutableStateFlow<UiState>(UiState.Idle)
    val state: StateFlow<UiState> = _uiState.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _state.value = UiState.Loading
            try {
                val users = repository.getUsers()
                _state.value = UiState.Success(users)
            } catch (e: Exception) {
                _state.value = UiState.Error(e.message)
            }
        }
    }
}
```

**Key Differences:**
- MVP: Manual view updates via callbacks
- MVVM: Automatic UI updates via data binding
- MVP: Presenter holds view reference
- MVVM: ViewModel has no view reference

### Q3: Explain Clean Architecture in Android

**Answer:**

Clean Architecture separates code into layers with dependencies pointing inward.

```kotlin
// Domain Layer (Business Logic)
data class User(val id: Int, val name: String)

interface UserRepository {
    suspend fun getUsers(): List<User>
}

class GetUsersUseCase(private val repository: UserRepository) {
    suspend operator fun invoke(): List<User> {
        return repository.getUsers()
    }
}

// Data Layer
class UserRepositoryImpl(
    private val api: UserApi,
    private val dao: UserDao
) : UserRepository {
    
    override suspend fun getUsers(): List<User> {
        val cached = dao.getAllUsers()
        if (cached.isNotEmpty()) return cached
        
        val remote = api.getUsers()
        dao.insertUsers(remote)
        return remote
    }
}

// Presentation Layer
class UserViewModel @Inject constructor(
    private val getUsersUseCase: GetUsersUseCase
) : ViewModel() {
    
    private val _state = MutableStateFlow<UiState>(UiState.Idle)
    val state: StateFlow<UiState> = _state.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _state.value = UiState.Loading
            try {
                val users = getUsersUseCase()
                _state.value = UiState.Success(users)
            } catch (e: Exception) {
                _state.value = UiState.Error(e.message)
            }
        }
    }
}
```

### Q4: What is the Repository pattern?

**Answer:**

Repository pattern abstracts data sources (API, Database, Cache).

```kotlin
// Interface
interface UserRepository {
    suspend fun getUsers(): Flow<List<User>>
    suspend fun getUserById(id: Int): User?
    suspend fun refresh()
}

// Implementation
class UserRepositoryImpl(
    private val api: UserApi,
    private val dao: UserDao,
    private val networkConnection: NetworkConnection
) : UserRepository {
    
    override suspend fun getUsers(): Flow<List<User>> {
        return flow {
            // Emit cached data first
            emit(dao.getAllUsers())
            
            // Try to fetch from network
            if (networkConnection.isAvailable()) {
                try {
                    val users = api.getUsers()
                    dao.insertUsers(users)
                } catch (e: Exception) {
                    // Return cached data on error
                }
            }
            
            // Emit updated cached data
            emit(dao.getAllUsers())
        }.flowOn(Dispatchers.IO)
    }
    
    override suspend fun getUserById(id: Int): User? {
        val cached = dao.getUserById(id)
        if (cached != null) return cached
        
        return api.getUserById(id)?.also { user ->
            dao.insertUser(user)
        }
    }
    
    override suspend fun refresh() {
        val users = api.getUsers()
        dao.clearAndInsert(users)
    }
}
```

### Q5: Explain SOLID principles with Android examples

**Answer:**

**S - Single Responsibility Principle**
```kotlin
// BAD: One class doing multiple things
class BadUserManager {
    fun validateEmail(email: String) { }
    fun saveUser(user: User) { }
    fun sendEmail(user: User) { }
    fun displayUser(user: User) { }
}

// GOOD: Separate responsibilities
class EmailValidator {
    fun validate(email: String): Boolean { }
}

class UserRepository {
    fun saveUser(user: User) { }
}

class EmailService {
    fun sendEmail(user: User) { }
}

class UserViewModel {
    fun displayUser(user: User) { }
}
```

**O - Open/Closed Principle**
```kotlin
// BAD: Modify existing class for new feature
class PaymentProcessor {
    fun processPayment(amount: Double, type: String) {
        when (type) {
            "credit" -> { }
            "debit" -> { }
            "paypal" -> { } // Had to modify
        }
    }
}

// GOOD: Open for extension, closed for modification
interface PaymentMethod {
    fun process(amount: Double)
}

class CreditCardPayment : PaymentMethod {
    override fun process(amount: Double) { }
}

class DebitCardPayment : PaymentMethod {
    override fun process(amount: Double) { }
}

class PaypalPayment : PaymentMethod {
    override fun process(amount: Double) { }
}
```

**L - Liskov Substitution Principle**
```kotlin
// BAD: Subclass violates parent behavior
open class View {
    open fun draw() { }
}

class Button : View() {
    override fun draw() {
        throw NotImplementedError() // Violates LSP
    }
}

// GOOD: Subclass maintains parent behavior
class Button : View() {
    override fun draw() {
        // Proper implementation
    }
}
```

**I - Interface Segregation Principle**
```kotlin
// BAD: Fat interface
interface DataSource {
    fun read()
    fun write()
    fun delete()
    fun upload() // Some classes don't need this
}

// GOOD: Segregated interfaces
interface Readable {
    fun read()
}

interface Writable {
    fun write()
}

interface Deletable {
    fun delete()
}

interface Uploadable {
    fun upload()
}

class CacheDataSource : Readable, Writable { }
class CloudDataSource : Readable, Writeable, Uploadable { }
```

**D - Dependency Inversion Principle**
```kotlin
// BAD: Depends on concrete class
class DataService {
    private val database = SqliteDatabase()
    
    fun getData() {
        database.query()
    }
}

// GOOD: Depends on abstraction
class DataService(private val database: Database) {
    fun getData() {
        database.query()
    }
}
```

### Q6: What is dependency injection and why is it important?

**Answer:**

Dependency Injection provides dependencies to classes rather than having classes create them.

```kotlin
// Without DI (Tight coupling)
class OrderViewModel {
    private val api = OrderApi()
    private val db = OrderDatabase()
    
    fun processOrder() {
        val orders = api.getOrders()
        db.saveOrders(orders)
    }
}

// With DI (Loose coupling)
class OrderViewModel(
    private val api: OrderApi,
    private val db: OrderDatabase
) {
    fun processOrder() {
        val orders = api.getOrders()
        db.saveOrders(orders)
    }
}

// Easy to test
class OrderViewModelTest {
    @Test
    fun testProcessOrder() {
        val mockApi = mockk<OrderApi>()
        val mockDb = mockk<OrderDatabase>()
        val viewModel = OrderViewModel(mockApi, mockDb)
        // Test viewModel
    }
}
```

### Q7: Explain LiveData vs Flow

**Answer:**

**LiveData:**
- Lifecycle-aware
- Simple API
- Automatic memory management

```kotlin
class UserViewModel : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    fun loadUsers() {
        viewModelScope.launch {
            val data = repository.getUsers()
            _users.value = data
        }
    }
}

// In Activity
viewModel.users.observe(this) { users ->
    // Update UI
}
```

**Flow:**
- More powerful
- Cold stream
- Rich operators

```kotlin
class UserViewModel : ViewModel() {
    val users: Flow<List<User>> = repository
        .getUsers()
        .flowOn(Dispatchers.IO)
        .catch { e -> emit(emptyList()) }
        .stateIn(
            viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
}

// In Activity
lifecycleScope.launch {
    viewModel.users.collect { users ->
        // Update UI
    }
}
```

### Q8: How does ViewModel work and when to use it?

**Answer:**

ViewModel is designed to store and manage UI-related data in a lifecycle-conscious way.

**Lifecycle:**
```kotlin
class MyViewModel : ViewModel() {
    
    init {
        println("ViewModel created")
    }
    
    override fun onCleared() {
        super.onCleared()
        println("ViewModel cleared")
        // Clean up resources
    }
}
```

**Usage:**
```kotlin
class MainActivity : AppCompatActivity() {
    
    // Get shared ViewModel
    private val viewModel: UserViewModel by viewModels()
    
    // Get activity-specific ViewModel
    private val localViewModel: UserViewModel by viewModels {
        ViewModelFactory()
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Observe ViewModel state
        lifecycleScope.launch {
            viewModel.state.collect { state ->
                // Update UI
            }
        }
    }
}
```

**When to Use:**
- Survive configuration changes
- Store UI-related data
- Keep data separate from Activity/Fragment
- Share data between fragments

### Q9: Explain multi-module architecture

**Answer:**

Split app into modules for better organization.

```
:app
  └── :core:ui
  └── :feature:auth
  └── :feature:home
  └── :feature:profile
```

```kotlin
// feature:auth
@AndroidEntryPoint
class LoginFragment : Fragment() {
    private val viewModel: LoginViewModel by viewModels()
}

class LoginViewModel @Inject constructor(
    private val authRepository: AuthRepository
) : ViewModel() { }

// feature:home
class HomeViewModel @Inject constructor(
    private val homeRepository: HomeRepository
) : ViewModel() { }
```

### Q10: How to implement offline-first architecture?

**Answer:**

**Strategy:**
1. Cache everything locally
2. Sync when online
3. Show cached data instantly

```kotlin
class OfflineFirstRepository(
    private val api: ApiService,
    private val database: AppDatabase
) {
    
    fun getData(): Flow<List<Data>> {
        return flow {
            // Emit cached data first
            emit(database.getAllData())
            
            // Try to fetch fresh data
            try {
                val freshData = api.getData()
                database.clearAndInsert(freshData)
                emit(freshData)
            } catch (e: Exception) {
                // Return cached data on error
            }
        }.flowOn(Dispatchers.IO)
    }
    
    fun syncData() {
        viewModelScope.launch {
            try {
                val data = api.getData()
                database.clearAndInsert(data)
            } catch (e: Exception) {
                // Handle error
            }
        }
    }
}
```

### Q11: What is the Single Source of Truth pattern?

**Answer:**

Single Source of Truth (SSOT) means having one authoritative source for data.

```kotlin
class SsotRepository(
    private val api: ApiService,
    private val cache: Cache
) {
    
    // Single source of truth: cache
    private val _data = MutableStateFlow<List<Item>>(emptyList())
    val data: StateFlow<List<Item>> = _data.asStateFlow()
    
    fun getData(): Flow<List<Item>> = flow {
        emit(_data.value)
    }
    
    suspend fun refresh() {
        val freshData = api.getData()
        cache.update(freshData)
        _data.value = freshData
    }
}
```

### Q12: Explain error handling in MVVM architecture

**Answer:**

**Sealed Classes for States:**
```kotlin
sealed class UiState<out T> {
    object Idle : UiState<Nothing>()
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
}

class UserViewModel : ViewModel() {
    private val _state = MutableStateFlow<UiState<List<User>>>(UiState.Idle)
    val state: StateFlow<UiState<List<User>>> = _state.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _state.value = UiState.Loading
            try {
                val users = repository.getUsers()
                _state.value = UiState.Success(users)
            } catch (e: NetworkException) {
                _state.value = UiState.Error("Network error")
            } catch (e: Exception) {
                _state.value = UiState.Error("Unknown error")
            }
        }
    }
}
```

---

**Previous:** [Java vs Kotlin](java-vs-kotlin.md)

**Next:** [Jetpack Compose](jetpack-compose.md)

