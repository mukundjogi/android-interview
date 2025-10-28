---
layout: default
title: "Kotlin Programming - Interview Preparation"
---

# Kotlin Programming

## Table of Contents
- [Null Safety](#null-safety)
- [Data Classes](#data-classes)
- [Sealed Classes](#sealed-classes)
- [Extension Functions](#extension-functions)
- [Scope Functions](#scope-functions)
- [Coroutines Basics](#coroutines-basics)
- [Inline Functions](#inline-functions)
- [Interview Questions & Answers](#interview-questions--answers)

## Null Safety

Kotlin's type system distinguishes between nullable and non-nullable types.

```kotlin
// Non-nullable
var name: String = "Kotlin"

// Nullable - must handle null
var nullableName: String? = null

// Safe call operator
val length = nullableName?.length // Returns null if nullableName is null

// Elvis operator
val result = nullableName?.length ?: 0 // Returns 0 if null

// Not-null assertion
val result2 = nullableName!!.length // Throws NPE if null
```

## Data Classes

Concise syntax for classes that hold data.

```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String
)

// Automatically generates:
// - equals(), hashCode(), toString()
// - copy() function
// - componentN() functions for destructuring

val user = User(1, "John", "john@example.com")
val copiedUser = user.copy(name = "Jane")

// Destructuring
val (id, name, email) = user
```

## Sealed Classes

Restrict class hierarchies to specific types.

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
    object Loading : Result<Nothing>()
}

fun handleResult(result: Result<User>) {
    when (result) {
        is Result.Success -> println("Success: ${result.data}")
        is Result.Error -> println("Error: ${result.message}")
        is Result.Loading -> println("Loading...")
    }
}
```

## Extension Functions

Add functionality to existing classes without modifying them.

```kotlin
// Extension function
fun String.isEmail(): Boolean {
    return this.contains("@") && this.contains(".")
}

// Usage
"test@example.com".isEmail()

// Extension on collections
fun <T> List<T>.secondOrNull(): T? = if (size >= 2) this[1] else null

// Extension with generic type
inline fun <reified T> Any.safeCast(): T? = this as? T
```

## Scope Functions

Functions that execute a block of code within an object's context.

```kotlin
data class Person(var name: String, var age: Int)

// let - returns last expression
val person = Person("John", 30)
val result = person.let {
    it.name = it.name.uppercase()
    it.age
} // result = 30

// apply - returns object
val person2 = Person("Jane", 25).apply {
    name = name.uppercase()
    age++
} // Returns Person object

// with - returns last expression
val result2 = with(person) {
    name = name.uppercase()
    age
}

// run - returns last expression
val result3 = person.run {
    name = name.uppercase()
    age
}

// also - returns object, for side effects
val person3 = person.also {
    println("Person created: ${it.name}")
}
```

## Coroutines Basics

Coroutines for asynchronous programming.

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Launch coroutine
        lifecycleScope.launch {
            val data = fetchData()
            updateUI(data)
        }
    }
    
    // Suspending function
    private suspend fun fetchData(): String {
        return withContext(Dispatchers.IO) {
            // Network call
            "Data"
        }
    }
}
```

## Inline Functions

Functions that avoid lambda overhead.

```kotlin
// Inline function
inline fun <T> Iterable<T>.forEach(action: (T) -> Unit) {
    for (element in this) action(element)
}

// Noinline - prevent inlining parameter
inline fun processData(
    crossinline callback: () -> Unit,
    noinline errorHandler: (Exception) -> Unit
) {
    try {
        callback()
    } catch (e: Exception) {
        errorHandler(e)
    }
}

// Reified type parameters
inline fun <reified T> Gson.fromJson(json: String): T {
    return fromJson(json, T::class.java)
}
```

## Null Safety

**What is Null Safety?**

Null safety is Kotlin's way of preventing the dreaded "NullPointerException" that crashes many apps. Think of it like having a safety net that catches you before you fall.

**Why Null Safety Matters:**

- **Prevents Crashes**: No more "app has stopped" errors from null values
- **Better Code**: Forces you to handle null cases properly
- **Compile-time Safety**: Catches null issues before your app runs
- **Cleaner Code**: Makes your intentions clear about what can be null

**How Null Safety Works:**

Kotlin's type system distinguishes between nullable and non-nullable types. It's like having two different types of boxes - one that can be empty (nullable) and one that always has something (non-nullable).

**Key Concepts:**

1. **Non-nullable Types**: Variables that can never be null
2. **Nullable Types**: Variables that can be null (marked with `?`)
3. **Safe Call Operator**: Safely access properties/methods on nullable objects
4. **Elvis Operator**: Provide default values when something is null
5. **Not-null Assertion**: Tell Kotlin you're sure something isn't null

```kotlin
// Non-nullable - this can never be null
var name: String = "Kotlin"

// Nullable - this can be null
var nullableName: String? = null

// Safe call operator - returns null if nullableName is null
val length = nullableName?.length // Returns null if nullableName is null

// Elvis operator - provides default value if null
val result = nullableName?.length ?: 0 // Returns 0 if null

// Not-null assertion - throws NPE if null (use carefully!)
val result2 = nullableName!!.length // Throws NPE if null

// Smart cast - Kotlin knows it's not null after null check
if (nullableName != null) {
    val length = nullableName.length // Smart cast to non-nullable
}
```

**How it works:**
1. `String` means "this can never be null"
2. `String?` means "this might be null"
3. `?.` safely accesses properties without crashing
4. `?:` provides a default value when something is null
5. `!!` tells Kotlin "I'm sure this isn't null" (dangerous!)

## Data Classes

**What are Data Classes?**

Data classes are Kotlin's way of creating classes that primarily hold data. Think of them as containers that automatically know how to store, compare, and copy their contents.

**Why Data Classes Matter:**

- **Less Boilerplate**: Automatically generate common methods
- **Cleaner Code**: Focus on data, not implementation details
- **Immutability**: Easy to create immutable data structures
- **Convenience**: Built-in methods for common operations

**What Data Classes Give You:**

1. **equals()**: Compare two objects for equality
2. **hashCode()**: Generate hash codes for collections
3. **toString()**: Convert object to readable string
4. **copy()**: Create copies with some properties changed
5. **componentN()**: Destructure objects into individual properties

```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String
)

// Automatically generates:
// - equals(), hashCode(), toString()
// - copy() function
// - componentN() functions for destructuring

val user = User(1, "John", "john@example.com")
val copiedUser = user.copy(name = "Jane")

// Destructuring - extract individual properties
val (id, name, email) = user
```

**How it works:**
1. `data class` tells Kotlin this is primarily for holding data
2. Kotlin automatically generates common methods
3. `copy()` lets you create new instances with some changes
4. Destructuring lets you extract individual properties

## Sealed Classes

**What are Sealed Classes?**

Sealed classes are like a restricted family tree - they define a limited set of possible types. Think of them as creating a closed set of options that can't be extended outside the file.

**Why Sealed Classes Matter:**

- **Type Safety**: Compiler ensures you handle all possible cases
- **Exhaustive When**: Forces you to handle every possible type
- **Pattern Matching**: Perfect for representing different states
- **Immutability**: Each subclass represents a specific case

**Common Use Cases:**

1. **API Responses**: Success, Error, Loading states
2. **UI States**: Different screen states
3. **Navigation**: Different screen types
4. **Result Types**: Success or failure outcomes

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
    object Loading : Result<Nothing>()
}

fun handleResult(result: Result<User>) {
    when (result) {
        is Result.Success -> println("Success: ${result.data}")
        is Result.Error -> println("Error: ${result.message}")
        is Result.Loading -> println("Loading...")
        // Compiler ensures you handle all cases!
    }
}
```

**How it works:**
1. `sealed class` creates a restricted hierarchy
2. All subclasses must be in the same file
3. `when` expressions must handle all possible cases
4. Compiler prevents you from missing any cases

## Extension Functions

**What are Extension Functions?**

Extension functions let you add new functionality to existing classes without modifying them. It's like adding new tools to an existing toolbox without changing the toolbox itself.

**Why Extension Functions Matter:**

- **Code Organization**: Group related functionality together
- **Readability**: Make code more expressive and readable
- **Reusability**: Use the same extension across your app
- **Clean APIs**: Create domain-specific methods

**How Extension Functions Work:**

```kotlin
// Extension function - adds functionality to String class
fun String.isEmail(): Boolean {
    return this.contains("@") && this.contains(".")
}

// Usage - now all Strings have isEmail() method
"test@example.com".isEmail() // true
"invalid".isEmail() // false

// Extension on collections
fun <T> List<T>.secondOrNull(): T? = if (size >= 2) this[1] else null

// Extension with generic type
inline fun <reified T> Any.safeCast(): T? = this as? T
```

**How it works:**
1. `fun ClassName.functionName()` defines the extension
2. `this` refers to the object the extension is called on
3. Extensions are resolved statically (at compile time)
4. They don't actually modify the original class

## Scope Functions

**What are Scope Functions?**

Scope functions are special functions that let you execute code within the context of an object. Think of them as temporary "workspaces" where you can work with an object.

**Why Scope Functions Matter:**

- **Code Organization**: Group related operations together
- **Readability**: Make code more expressive
- **Null Safety**: Safe operations on nullable objects
- **Functional Style**: Write more functional code

**The Five Scope Functions:**

1. **let** - Returns last expression, uses `it`
2. **run** - Returns last expression, uses `this`
3. **with** - Returns last expression, uses `this` (not extension)
4. **apply** - Returns object, uses `this`
5. **also** - Returns object, uses `it`

```kotlin
data class Person(var name: String, var age: Int)

// let - returns last expression
val person = Person("John", 30)
val result = person.let {
    it.name = it.name.uppercase()
    it.age
} // result = 30

// apply - returns object
val person2 = Person("Jane", 25).apply {
    name = name.uppercase()
    age++
} // Returns Person object

// with - returns last expression
val result2 = with(person) {
    name = name.uppercase()
    age
}

// run - returns last expression
val result3 = person.run {
    name = name.uppercase()
    age
}

// also - returns object, for side effects
val person3 = person.also {
    println("Person created: ${it.name}")
}
```

**When to Use Each:**

- **let**: Transform nullable objects safely
- **run**: Group operations on an object
- **with**: Group operations (not extension function)
- **apply**: Configure object properties
- **also**: Perform side effects

## Coroutines Basics

**What are Coroutines?**

Coroutines are Kotlin's way of handling asynchronous programming. Think of them as lightweight threads that can pause and resume their work without blocking the main thread.

**Why Coroutines Matter:**

- **Non-blocking**: Don't freeze the UI while waiting
- **Lightweight**: Thousands of coroutines can run on one thread
- **Structured**: Easy to manage and cancel
- **Sequential**: Code looks like regular sequential code

**Key Concepts:**

1. **Suspend Functions**: Functions that can pause execution
2. **Coroutine Scope**: Defines the lifetime of coroutines
3. **Dispatchers**: Determine which thread coroutines run on
4. **Structured Concurrency**: Parent coroutines control child coroutines

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Launch coroutine
        lifecycleScope.launch {
            val data = fetchData()
            updateUI(data)
        }
    }
    
    // Suspending function - can pause execution
    private suspend fun fetchData(): String {
        return withContext(Dispatchers.IO) {
            // Network call - this won't block the UI
            "Data"
        }
    }
}
```

**How it works:**
1. `lifecycleScope.launch` starts a coroutine
2. `suspend fun` can pause without blocking
3. `withContext(Dispatchers.IO)` switches to background thread
4. UI thread stays responsive while data loads

## Inline Functions

**What are Inline Functions?**

Inline functions are functions that are expanded at the call site, eliminating the overhead of function calls. Think of them as macros that copy the function code directly where it's called.

**Why Inline Functions Matter:**

- **Performance**: Eliminate function call overhead
- **Lambda Optimization**: Avoid creating lambda objects
- **Reified Types**: Access generic type information at runtime
- **Control Flow**: Use return statements in lambdas

**How Inline Functions Work:**

```kotlin
// Inline function - code is copied to call site
inline fun <T> Iterable<T>.forEach(action: (T) -> Unit) {
    for (element in this) action(element)
}

// Noinline - prevent inlining parameter
inline fun processData(
    crossinline callback: () -> Unit,
    noinline errorHandler: (Exception) -> Unit
) {
    try {
        callback()
    } catch (e: Exception) {
        errorHandler(e)
    }
}

// Reified type parameters - access type at runtime
inline fun <reified T> Gson.fromJson(json: String): T {
    return fromJson(json, T::class.java)
}
```

**How it works:**
1. `inline` tells compiler to copy function code
2. `noinline` prevents a parameter from being inlined
3. `crossinline` allows return statements in lambdas
4. `reified` lets you access generic type information

## Interview Questions & Answers

### Q1: What is null safety in Kotlin and how does it prevent NullPointerException?

**Answer:**

Kotlin's type system distinguishes nullable and non-nullable types to prevent NPE at compile time.

```kotlin
// Non-nullable
var name: String = "Kotlin"

// Nullable
var nullableName: String? = null

// Safe call - returns null if left side is null
val length = nullableName?.length // null

// Elvis operator - provides default value
val result = nullableName?.length ?: 0 // 0

// Not-null assertion - throws NPE if null
val result2 = nullableName!!.length // Throws NPE

// Smart cast
if (nullableName != null) {
    val length = nullableName.length // Smart cast to non-nullable
}
```

### Q2: What are data classes and what benefits do they provide?

**Answer:**

Data classes are classes that primarily hold data with auto-generated boilerplate code.

```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String
)

// Automatically generated:
// equals(), hashCode(), toString(), copy(), componentN()

val user1 = User(1, "John", "john@example.com")
val user2 = User(1, "John", "john@example.com")

println(user1 == user2) // true (equals())
println(user1.toString()) // "User(id=1, name=John, email=john@example.com)"

// Copy with modification
val modifiedUser = user1.copy(name = "Jane")

// Destructuring
val (id, name, email) = user1
```

### Q3: Explain sealed classes in Kotlin and provide use cases.

**Answer:**

Sealed classes restrict subclass inheritance to specific types.

```kotlin
sealed class UiState {
    data class Success(val data: String) : UiState()
    data class Error(val message: String) : UiState()
    object Loading : UiState()
}

fun handleState(state: UiState) {
    when (state) {
        is UiState.Success -> println("Data: ${state.data}")
        is UiState.Error -> println("Error: ${state.message}")
        is UiState.Loading -> println("Loading...")
    }
}
```

### Q4: What are extension functions and when should you use them?

**Answer:**

Extension functions allow adding functionality to existing classes.

```kotlin
// Extend String class
fun String.isEmail(): Boolean {
    return contains("@") && contains(".")
}

// Usage
if ("test@example.com".isEmail()) {
    println("Valid email")
}

// Extension on collections
fun <T> List<T>.secondOrNull(): T? = if (size >= 2) this[1] else null
```

### Q5: Explain scope functions (let, run, with, apply, also) with examples.

**Answer:**

Scope functions execute code in an object's context.

```kotlin
data class Person(var name: String, var age: Int)

// let - returns last expression
val result = Person("John", 30).let {
    it.name.uppercase()
} // "JOHN"

// run - returns last expression, uses 'this'
val result2 = Person("John", 30).run {
    name.uppercase()
} // "JOHN"

// apply - returns object, uses 'this'
val person = Person("John", 30).apply {
    name = name.uppercase()
} // Returns Person object

// also - returns object, for side effects
val person2 = Person("John", 30).also {
    println("Person created: ${it.name}")
}

// with - non-extension version
val result3 = with(Person("John", 30)) {
    name.uppercase()
}
```

### Q6: What are coroutines and how do they differ from threads?

**Answer:**

Coroutines are lightweight threads for asynchronous programming.

```kotlin
suspend fun fetchUser(): User {
    return withContext(Dispatchers.IO) {
        // Network call
        apiService.getUser()
    }
}

// Usage
viewModelScope.launch {
    val user = fetchUser()
    updateUI(user)
}
```

**Differences:**
- Threads: OS-level, heavy weight
- Coroutines: User-level, lightweight
- Coroutines can be cancelled
- Coroutines use structured concurrency

### Q7: What are inline functions and when to use them?

**Answer:**

Inline functions are expanded at call site to avoid lambda overhead.

```kotlin
// Inline function
inline fun <T> Iterable<T>.forEach(action: (T) -> Unit) {
    for (element in this) action(element)
}

// Reified type parameters
inline fun <reified T> createInstance(): T {
    return T::class.constructors.first().call()
}

// Usage eliminates lambda allocation overhead
```

### Q8: Explain type system in Kotlin (Any, Nothing, Unit).

**Answer:**

```kotlin
// Any - root of all classes
fun process(value: Any) {
    when (value) {
        is String -> println(value.length)
        is Int -> println(value * 2)
    }
}

// Nothing - used for functions that never return
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}

// Unit - equivalent to void in Java
fun printMessage(message: String): Unit {
    println(message)
}

// Inferred return type is Unit
fun printMessage(message: String) {
    println(message)
}
```

---

## Navigation

**Previous:** [Android Basics](android-basics.md)

**Next:** [Java vs Kotlin](java-vs-kotlin.md)

