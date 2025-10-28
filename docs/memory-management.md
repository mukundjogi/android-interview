---
layout: default
title: "Memory Management - Interview Preparation"
---

# Memory Management

## Table of Contents
- [Memory Model](#memory-model)
- [Garbage Collection](#garbage-collection)
- [Memory Leaks](#memory-leaks)
- [LeakCanary](#leakcanary)
- [Interview Questions & Answers](#interview-questions--answers)

## Memory Leaks

```kotlin
// Memory leak example
class MainActivity : AppCompatActivity() {
    
    // LEAK: Activity holds reference to singleton
    private val leakyListener = object : Listener {
        override fun onEvent(data: String) {
            // Activity is captured here
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        GlobalManager.register(leakyListener) // Leak!
    }
}

// Solution: Use WeakReference or unregister
class MainActivity : AppCompatActivity() {
    
    private val listener = object : Listener {
        override fun onEvent(data: String) {
            // Handle event
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        GlobalManager.register(listener)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        GlobalManager.unregister(listener) // Prevent leak
    }
}
```

## Memory Model

**What is Android's Memory Model?**

Android uses a memory model based on the Java Virtual Machine (JVM), but with some Android-specific modifications. Think of memory as different "rooms" in a house, each serving a specific purpose.

**Why Understanding Memory Matters:**

- **Performance**: Proper memory usage makes your app faster
- **Stability**: Prevents crashes from running out of memory
- **User Experience**: Smooth app performance
- **Battery Life**: Efficient memory usage saves battery

**Memory Areas Explained:**

1. **Heap Memory** - The "Storage Room"
   - Where all your objects live (like `ArrayList`, `String`, custom classes)
   - Managed by Garbage Collector
   - Can grow and shrink as needed
   - Shared between all threads in your app

2. **Stack Memory** - The "Work Desk"
   - Where method calls and local variables live
   - Each thread has its own stack
   - Very fast access
   - Automatically cleaned up when method ends

3. **Native Heap** - The "Special Storage"
   - For C/C++ code and native libraries
   - Not managed by Java Garbage Collector
   - Used by Android system and native libraries

```kotlin
class MemoryExample {
    // This object lives in HEAP memory
    val objectOnHeap = ArrayList<String>()
    
    fun methodOnStack() {
        // This variable lives in STACK memory
        val localVariable = 42
        
        // When method ends, localVariable is automatically removed
        // But objectOnHeap stays in heap until garbage collected
    }
}
```

**How Memory Works:**
1. When you create an object, it goes to the heap
2. Local variables go to the stack
3. When a method ends, stack variables are removed
4. Heap objects stay until no longer referenced
5. Garbage Collector removes unused heap objects

## Garbage Collection

**What is Garbage Collection?**

Garbage Collection (GC) is Android's automatic "cleanup crew" for memory. It's like having a robot that automatically throws away things you're no longer using, so you don't run out of space.

**Why Garbage Collection Matters:**

- **Automatic Cleanup**: You don't need to manually free memory
- **Prevents Memory Leaks**: Removes objects that are no longer referenced
- **Prevents Crashes**: Avoids "Out of Memory" errors
- **Performance**: Keeps your app running smoothly

**How Garbage Collection Works:**

Think of it like cleaning your room:
1. **Mark Phase**: GC looks at all objects and marks which ones are still being used
2. **Sweep Phase**: GC removes (deletes) objects that aren't marked as used
3. **Compact Phase**: GC reorganizes remaining objects to reduce fragmentation

**Types of Garbage Collectors:**

- **Concurrent GC**: Runs alongside your app (less interruption)
- **Parallel GC**: Uses multiple threads for faster collection
- **G1 GC**: More advanced, handles large heaps better

**When GC Runs:**
- When heap memory gets full
- When system is low on memory
- Periodically to keep things clean

## Memory Leaks

**What is a Memory Leak?**

A memory leak is like forgetting to turn off a faucet - water (memory) keeps flowing even when you don't need it anymore. In programming, it means objects that should be deleted are still taking up memory.

**Why Memory Leaks are Bad:**

- **App Slowdown**: Less available memory makes everything slower
- **Crashes**: Eventually leads to "Out of Memory" errors
- **Battery Drain**: More memory usage = more battery consumption
- **Poor User Experience**: App becomes unresponsive

**Common Causes of Memory Leaks:**

1. **Long-lived References**: Objects that live longer than they should
2. **Unregistered Listeners**: Not removing event listeners
3. **Static References**: Static variables holding object references
4. **Context Leaks**: Using Activity context in long-lived objects

```kotlin
// Memory leak example
class MainActivity : AppCompatActivity() {
    
    // LEAK: Activity holds reference to singleton
    private val leakyListener = object : Listener {
        override fun onEvent(data: String) {
            // Activity is captured here
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        GlobalManager.register(leakyListener) // Leak!
    }
}

// Solution: Use WeakReference or unregister
class MainActivity : AppCompatActivity() {
    
    private val listener = object : Listener {
        override fun onEvent(data: String) {
            // Handle event
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        GlobalManager.register(listener)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        GlobalManager.unregister(listener) // Prevent leak
    }
}
```

**How to Prevent Memory Leaks:**

1. **Use Application Context**: For long-lived objects
2. **Unregister Listeners**: Always clean up in `onDestroy()`
3. **Avoid Static References**: Don't hold object references in static variables
4. **Use WeakReference**: For references that can be garbage collected
5. **Cancel Background Work**: Stop coroutines, threads, and timers

## LeakCanary

**What is LeakCanary?**

LeakCanary is like a detective that automatically finds memory leaks in your app during development. It watches for objects that should be garbage collected but aren't, indicating a memory leak.

**Why Use LeakCanary?**

- **Automatic Detection**: Finds leaks without manual testing
- **Detailed Reports**: Shows exactly what's causing the leak
- **Quick Feedback**: Immediate notification when leaks are found
- **Development Only**: Automatically disabled in production builds

**How LeakCanary Works:**

1. **Watch Objects**: Monitors Activities, Fragments, and other components
2. **Wait for Destruction**: Waits for objects to be destroyed
3. **Trigger GC**: Forces garbage collection after a delay
4. **Check Retention**: Sees if objects are still in memory
5. **Report Leaks**: Shows detailed leak reports if objects are retained

**Setting Up LeakCanary:**

```gradle
dependencies {
    debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.12'
}
```

**What LeakCanary Reports:**

- **Leaked Object**: What object is leaking
- **Retention Path**: How the object is being held in memory
- **Suggested Fix**: How to fix the leak

## Interview Questions & Answers

### Q1: Explain Android memory model

**Answer:**

Android uses Java Virtual Machine (JVM) memory model with separate heaps for apps.

**Memory Areas:**
- Heap: Objects live here
- Stack: Method calls and local variables
- Native heap: C/C++ allocations

```kotlin
class MemoryExample {
    val objectOnHeap = ArrayList<String>()
    
    fun methodOnStack() {
        val localVariable = 42
    }
}
```

### Q2: How does garbage collection work on Android?

**Answer:**

**Conceptual Understanding:**

Garbage Collection (GC) is Android's automatic memory management system. It identifies and reclaims memory from objects that are no longer in use. Understanding GC helps optimize app performance and avoid memory issues.

**The Garbage Collection Cycle:**

The GC process involves identifying which objects are still being used (reachable) and which are not (unreachable), then freeing memory from unreachable objects.

**GC Process:**
1. Mark: Mark all reachable objects
2. Sweep: Remove unreachable objects
3. Compact: Defragment memory

**Types of GC:**
- Concurrent GC (CMS)
- Parallel GC
- G1 GC

### Q3: How to prevent memory leaks?

**Answer:**

**Conceptual Understanding:**

Memory leaks occur when objects are no longer needed but still referenced, preventing garbage collection. In Android, common causes include references to Activities, unclosed resources, and background work holding references.

**Why Leaks Matter:**
- App slowdowns and crashes
- OutOfMemoryErrors
- Poor user experience
- Battery drain

**Detection:**
- Use LeakCanary for automatic detection
- Monitor heap in Android Studio Profiler
- Watch for memory growth over time

**Common Leak Causes:**
- Long-lived references holding Activity context
- Unregistered listeners
- Async tasks without proper cancellation

**Solutions:**
```kotlin
// Use application context for long-lived objects
class DataManager(context: Context) {
    private val appContext = context.applicationContext
}

// Unregister listeners
override fun onDestroy() {
    super.onDestroy()
    EventBus.unregister(this)
}

// Cancel coroutines
override fun onDestroy() {
    super.onDestroy()
    job?.cancel()
}
```

### Q4: How to optimize bitmap memory usage?

**Answer:**

**Conceptual Understanding:**

Bitmaps are one of the largest memory consumers in Android apps. A single full-screen bitmap can consume several megabytes of RAM. Proper bitmap handling is critical for app performance.

**Memory Calculation:**
Memory = width × height × bytes per pixel

For example: 1920×1080×4 = 8MB per bitmap

**Optimization Strategies:**
- Load appropriately sized bitmaps
- Recycle bitmaps when done
- Use appropriate bitmaps formats
- Cache wisely

```kotlin
fun decodeSampledBitmap(resources: Resources, resId: Int, reqWidth: Int, reqHeight: Int): Bitmap {
    return BitmapFactory.Options().run {
        inJustDecodeBounds = true
        BitmapFactory.decodeResource(resources, resId, this)
        
        inSampleSize = calculateInSampleSize(this, reqWidth, reqHeight)
        
        inJustDecodeBounds = false
        BitmapFactory.decodeResource(resources, resId, this)
    }
}
```

### Q5: How does LeakCanary work?

**Answer:**

**Conceptual Understanding:**

LeakCanary is an automated memory leak detection tool for Android. It automatically watches for objects that should be garbage collected but aren't, indicating a memory leak.

**Detection Mechanism:**

The tool automatically tracks certain Android lifecycle components (Activities, Fragments, etc.) and checks if they're properly garbage collected after being destroyed.

**The Process:**
1. Watch destroyed objects
2. Trigger garbage collection after a delay
3. Check if objects are still retained in memory
4. Analyze the retaining path to find the leak

**Benefits:**
- Automatic detection during development
- Detailed leak reports
- Quick feedback loop
- No impact on production (disabled in release builds)

LeakCanary detects memory leaks by:
1. Watching destroyed objects
2. Triggering GC after delay
3. Checking if objects are still retained
4. Reporting retained objects

```gradle
dependencies {
    debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.12'
}
```

### Q6: What tools to use for memory profiling?

**Answer:**

**Conceptual Overview:**

Memory profiling tools help you understand how your app uses memory, identify leaks, and optimize memory consumption. Different tools serve different purposes.

**Profiling Techniques:**
- Real-time heap monitoring
- Object allocation tracking
- GC event tracking
- Heap dump analysis
- Leak detection

**Tool Categories:**
1. Integrated tools (Android Studio Profiler)
2. Automatic leak detection (LeakCanary)
3. Advanced analysis (MAT, Perfetto)

**Android Studio Profiler:**
- Memory Profiler
- Heap Dump
- Allocation Tracking

**LeakCanary:**
- Automatic leak detection

**MAT (Memory Analyzer Tool):**
- Detailed heap analysis

### Q7: Explain WeakReference, SoftReference, and PhantomReference

**Answer:**

**Conceptual Foundation:**

References in Java/Kotlin determine when objects become eligible for garbage collection. Understanding different reference types helps manage memory efficiently and prevent memory leaks.

**Reference Strength Hierarchy:**
Strong > Soft > Weak > Phantom

**Use Cases:**
- **WeakReference**: For caching that should be GC'd when memory is needed
- **SoftReference**: For caches that should persist longer
- **PhantomReference**: For cleanup tracking (rarely used in Android)

**WeakReference:**
```kotlin
val weakRef = WeakReference(object)
weakRef.get() // May return null
```

**SoftReference:**
```kotlin
val softRef = SoftReference(object)
softRef.get() // Cleared when memory pressure
```

**PhantomReference:**
```kotlin
val phantomRef = PhantomReference(object, referenceQueue)
// Used for cleanup tracking
```

### Q8: How to profile memory in Android?

**Answer:**

Leverage Android Studio Profiler to inspect allocations, leaks, and heap usage in real time.
Capture heap dumps and use dumpsys for snapshots to diagnose problematic objects.

**Using Android Studio Profiler:**
1. Run app in debug mode
2. Open Profiler window
3. Select Memory tab
4. Monitor heap allocation
5. Trigger GC manually
6. Capture heap dump

**Command Line:**
```bash
adb shell dumpsys meminfo <package_name>
```

---

**Previous:** [Dependency Injection](dependency-injection.md)

**Next:** [Testing](testing.md)

