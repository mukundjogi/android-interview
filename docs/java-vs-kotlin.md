---
layout: default
title: "Java vs Kotlin - Interview Preparation"
---

# Java vs Kotlin

## Table of Contents
- [Why Kotlin?](#why-kotlin)
- [Syntax Comparison](#syntax-comparison)
- [Null Safety](#null-safety)
- [Interoperability](#interoperability)
- [Interview Questions & Answers](#interview-questions--answers)

## Why Kotlin?

Kotlin is officially supported by Google for Android development and offers modern language features.

### Key Advantages

1. **Null Safety** - Prevents NullPointerException
2. **Conciseness** - Less boilerplate code
3. **Interoperability** - Works with existing Java code
4. **Modern Features** - Coroutines, extension functions, data classes
5. **Better IDE Support** - Strong tooling

## Syntax Comparison

### Classes

**Java:**
```java
public class User {
    private String name;
    private int age;
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    // equals, hashCode, toString must be manual
}
```

**Kotlin:**
```kotlin
data class User(
    val name: String,
    var age: Int
)
```

### Null Safety

**Java:**
```java
String name = null;
int length = name.length(); // NPE
```

**Kotlin:**
```kotlin
var name: String? = null
val length = name?.length // Safe, returns null
```

### Extension Functions

**Java:**
```java
public class StringUtils {
    public static boolean isEmail(String text) {
        return text.contains("@") && text.contains(".");
    }
}
```

**Kotlin:**
```kotlin
fun String.isEmail(): Boolean {
    return contains("@") && contains(".")
}
```

## Interview Questions & Answers

### Q1: Why is Kotlin preferred over Java for Android development?

**Answer:**

**Official Android Language**: Google announced Kotlin as first-class language for Android at Google I/O 2017.

**Key Reasons:**
1. **Null Safety**: Compile-time null checking prevents NPE
2. **Less Boilerplate**: Reduces code by ~40%
3. **Modern Features**: Coroutines, data classes, extension functions
4. **100% Interop**: Works with existing Java codebase
5. **Better IDE Support**: IntelliJ IDEA provides excellent Kotlin support

```kotlin
// Kotlin: Concise
data class User(val name: String, val age: Int)

// Java: Verbose
public class User {
    private String name;
    private int age;
    // ... constructors, getters, setters, equals, hashCode, toString
}
```

### Q2: What are the main syntax differences between Java and Kotlin?

**Answer:**

**Key Differences:**

**1. Type Inference**
```kotlin
// Kotlin
val name = "Kotlin" // Type inferred
var age = 25

// Java
String name = "Java";
int age = 25;
```

**2. String Interpolation**
```kotlin
// Kotlin
val message = "Hello, $name"
val fullMessage = "Full: ${name.uppercase()}"

// Java
String message = "Hello, " + name;
String fullMessage = "Full: " + name.toUpperCase();
```

**3. When vs Switch**
```kotlin
// Kotlin
when (status) {
    "active" -> start()
    "inactive" -> stop()
    else -> suspend()
}

// Java
switch (status) {
    case "active":
        start();
        break;
    case "inactive":
        stop();
        break;
    default:
        suspend();
}
```

### Q3: How does Kotlin handle null safety compared to Java?

**Answer:**

**Java:**
```java
String name = getData(); // May be null
int length = name.length(); // NPE risk
if (name != null) {
    length = name.length(); // Manual check
}
```

**Kotlin:**
```kotlin
val name: String? = getData()
val length = name?.length // Safe call returns null
val length2 = name?.length ?: 0 // Elvis operator
val length3 = name!!.length // Not-null assertion (throws if null)

// Smart cast
if (name != null) {
    val length = name.length // Smart cast to non-null
}
```

### Q4: Explain extension functions in Kotlin and how they compare to Java

**Answer:**

**Java - Utility Classes:**
```java
public class StringUtils {
    public static boolean isEmail(String text) {
        return text.contains("@") && text.contains(".");
    }
    
    public static String capitalize(String text) {
        if (text == null || text.isEmpty()) return text;
        return text.substring(0, 1).toUpperCase() + text.substring(1);
    }
}

// Usage
StringUtils.isEmail("test@example.com");
```

**Kotlin - Extension Functions:**
```kotlin
fun String.isEmail(): Boolean {
    return contains("@") && contains(".")
}

fun String.capitalize(): String {
    return if (isEmpty()) this else "${this[0].uppercase()}${substring(1)}"
}

// Usage
"test@example.com".isEmail()
"hello".capitalize()
```

**Benefits:**
- More readable code
- Type-safe
- No utility class needed
- Works with existing classes

### Q5: What are data classes and how do they compare to Java POJOs?

**Answer:**

**Java POJO:**
```java
public class User {
    private String name;
    private int age;
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return age == user.age && Objects.equals(name, user.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    
    @Override
    public String toString() {
        return "User{name='" + name + "', age=" + age + '}';
    }
}
```

**Kotlin Data Class:**
```kotlin
data class User(
    val name: String,
    val age: Int
)
```

**Auto-generated:**
- `equals()`
- `hashCode()`
- `toString()`
- `copy()`
- `componentN()`

### Q6: How does Kotlin interop with Java?

**Answer:**

**100% Interoperability:**

```kotlin
// Calling Java from Kotlin
class KotlinClass {
    fun useJavaClass() {
        val javaClass = JavaClass()
        javaClass.javaMethod()
    }
}

// Calling Kotlin from Java
// Kotlin file
object KotlinUtils {
    @JvmStatic
    fun kotlinFunction() {
        // Implementation
    }
}

// Java file
class JavaClass {
    void callKotlin() {
        KotlinUtils.INSTANCE.kotlinFunction();
    }
}
```

**Annotations for Interop:**
```kotlin
@JvmName("customName")
fun functionName() { }

@JvmStatic
fun staticFunction() { }

@JvmOverloads
fun functionWithDefaults(param: String = "default") { }
```

### Q7: Explain coroutines vs Java threads

**Answer:**

**Java Threading:**
```java
// Creating threads
Thread thread = new Thread(() -> {
    // Background work
    String result = fetchData();
    runOnUiThread(() -> {
        updateUI(result);
    });
});
thread.start();

// Using ExecutorService
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> {
    String result = fetchData();
    new Handler(Looper.getMainLooper()).post(() -> {
        updateUI(result);
    });
});
```

**Kotlin Coroutines:**
```kotlin
// Lightweight, structured concurrency
viewModelScope.launch {
    val result = withContext(Dispatchers.IO) {
        fetchData()
    }
    // Automatically on main thread
    updateUI(result)
}

// Suspend function
suspend fun fetchData(): String = withContext(Dispatchers.IO) {
    delay(1000)
    "Data"
}
```

**Advantages of Coroutines:**
- Lightweight (not OS-level threads)
- Easier to cancel
- Structured concurrency
- No callback hell
- Better for Android lifecycle

### Q8: What are sealed classes and how do they differ from enums?

**Answer:**

**Java Enums:**
```java
public enum Status {
    LOADING,
    SUCCESS,
    ERROR;
    
    public void doSomething() {
        // Limited functionality
    }
}
```

**Kotlin Sealed Classes:**
```kotlin
sealed class Result<out T> {
    object Loading : Result<Nothing>()
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
}

// Usage
fun handleResult(result: Result<User>) {
    when (result) {
        is Result.Loading -> showLoader()
        is Result.Success -> displayData(result.data)
        is Result.Error -> showError(result.message)
    }
}
```

**Differences:**
- Sealed classes can have properties
- Sealed classes can be data classes
- Sealed classes support inheritance
- Enums are constants only

### Q9: How to migrate from Java to Kotlin?

**Answer:**

**Migration Strategy:**

**1. Start with New Features**
```kotlin
// Write new code in Kotlin
class NewFeature : ViewModel() {
    // Kotlin code
}
```

**2. Migrate Utilities First**
```kotlin
// Convert utility classes
object StringUtils {
    fun isEmail(text: String): Boolean {
        return text.contains("@") && text.contains(".")
    }
}
```

**3. Migrate Data Models**
```kotlin
// Easy migration: POJOs to data classes
data class User(
    val name: String,
    val email: String
)
```

**4. Use Java Interop**
```kotlin
// Kotlin can call Java
val javaService = JavaService()
val result = javaService.method()
```

**IntelliJ Conversion:**
- Code → Convert Java File to Kotlin
- Automatic conversion with manual fixes needed

### Q10: Compare performance of Kotlin vs Java

**Answer:**

**Compilation:**
- Kotlin: Slightly slower (due to type inference)
- Java: Faster compilation
- Difference is negligible in most cases

**Runtime Performance:**
- Kotlin: Compiles to same bytecode
- Java: Native Java bytecode
- Performance is essentially identical

**Code Size:**
- Kotlin: More concise, smaller bytecode
- Java: More verbose
- Kotlin ~30-40% less code

**Memory Usage:**
- Similar memory footprint
- Kotlin null safety reduces NPE exceptions

---

**Previous:** [Kotlin Programming](kotlin-programming.md)

**Next:** [Android Architecture](android-architecture.md)

