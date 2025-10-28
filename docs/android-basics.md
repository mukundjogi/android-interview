---
layout: default
title: "Android Basics - Interview Preparation"
---

# Android Basics

## Table of Contents
- [Activity Lifecycle](#activity-lifecycle)
- [Fragment Lifecycle](#fragment-lifecycle)
- [Application Lifecycle](#application-lifecycle)
- [Intents and Intent Filters](#intents-and-intent-filters)
- [Permissions](#permissions)
- [Context Types](#context-types)
- [Configuration Changes](#configuration-changes)
- [Interview Questions & Answers](#interview-questions--answers)

## Activity Lifecycle

Activities are the entry points for user interaction in Android apps. Understanding their lifecycle is crucial.

### Lifecycle States

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Initialize activity
        // Set content view
        // Setup initial state
    }
    
    override fun onStart() {
        super.onStart()
        // Activity becomes visible
        // Resume UI updates
    }
    
    override fun onResume() {
        super.onResume()
        // Activity is in foreground
        // Start animations, sensors
    }
    
    override fun onPause() {
        super.onPause()
        // Activity is partially visible
        // Pause animations, sensors
    }
    
    override fun onStop() {
        super.onStop()
        // Activity is no longer visible
        // Stop intensive operations
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // Activity is being destroyed
        // Release resources
    }
    
    override fun onRestart() {
        super.onRestart()
        // Activity restarting from stopped state
    }
}
```

### Lifecycle Aware Components

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Observe lifecycle events
        lifecycle.addObserver(object : LifecycleObserver {
            @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
            fun resume() {
                // Handle resume
            }
            
            @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
            fun pause() {
                // Handle pause
            }
        })
    }
}
```

## Fragment Lifecycle

Fragments have a more complex lifecycle tied to their host activity.

```kotlin
class MyFragment : Fragment() {
    
    override fun onAttach(context: Context) {
        super.onAttach(context)
        // Fragment attached to activity
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Fragment creation
    }
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate layout
        return inflater.inflate(R.layout.fragment_main, container, false)
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // View fully created, safe to access views
    }
    
    override fun onStart() {
        super.onStart()
        // Fragment visible
    }
    
    override fun onResume() {
        super.onResume()
        // Fragment active
    }
    
    override fun onPause() {
        super.onPause()
        // Fragment paused
    }
    
    override fun onStop() {
        super.onStop()
        // Fragment stopped
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        // View destroyed, but fragment still exists
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // Fragment destroyed
    }
    
    override fun onDetach() {
        super.onDetach()
        // Fragment detached from activity
    }
}
```

## Application Lifecycle

Understanding how your app lifecycle works.

```kotlin
class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        // App created
        initializeApp()
    }
    
    private fun initializeApp() {
        // Setup app-wide components
    }
    
    override fun onLowMemory() {
        super.onLowMemory()
        // System is low on memory
        // Release resources
    }
    
    override fun onTrimMemory(level: Int) {
        super.onTrimMemory(level)
        // Memory trimming requested
        when (level) {
            TRIM_MEMORY_RUNNING_CRITICAL -> {
                // Release non-critical resources
            }
            TRIM_MEMORY_UI_HIDDEN -> {
                // Release UI resources
            }
        }
    }
    
    override fun onConfigurationChanged(newConfig: Configuration) {
        super.onConfigurationChanged(newConfig)
        // Configuration changed
    }
}
```

## Intents and Intent Filters

Intents are used for communication between components.

```kotlin
// Explicit Intent
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("key", "value")
startActivity(intent)

// Implicit Intent
val intent = Intent(Intent.ACTION_VIEW)
intent.data = Uri.parse("https://www.example.com")
startActivity(intent)

// Intent with result
val intent = Intent(this, SelectionActivity::class.java)
startActivityForResult(intent, REQUEST_CODE)
```

### Handling Intent Results

```kotlin
override fun onActivityResult(
    requestCode: Int,
    resultCode: Int,
    data: Intent?
) {
    super.onActivityResult(requestCode, resultCode, data)
    
    if (requestCode == REQUEST_CODE && resultCode == RESULT_OK) {
        val result = data?.getStringExtra("result")
        // Handle result
    }
}
```

### Intent Filters

```xml
<activity android:name=".ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

## Permissions

Android uses a permission system to protect user privacy.

### Runtime Permissions

```kotlin
class MainActivity : AppCompatActivity() {
    
    private val permissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        if (isGranted) {
            // Permission granted
        } else {
            // Permission denied
        }
    }
    
    fun requestPermission() {
        if (ContextCompat.checkSelfPermission(
                this,
                Manifest.permission.CAMERA
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            permissionLauncher.launch(Manifest.permission.CAMERA)
        }
    }
}
```

### Multiple Permissions

```kotlin
private val requestMultiplePermissions = 
    registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { permissions ->
        permissions.entries.forEach { entry ->
            if (entry.value) {
                // Permission granted
            } else {
                // Permission denied
            }
        }
    }

fun requestMultiplePermissions() {
    requestMultiplePermissions.launch(
        arrayOf(
            Manifest.permission.CAMERA,
            Manifest.permission.READ_EXTERNAL_STORAGE
        )
    )
}
```

## Context Types

Understanding different context types and when to use them.

```kotlin
class MainActivity : AppCompatActivity() {
    
    // Application Context - Use for app-wide operations
    private val appContext: Context = applicationContext
    
    // Activity Context - Use for UI operations
    private val activityContext: Context = this
    
    // View Context - Bound to View's lifetime
    private fun getViewContext(): Context {
        return view.context // where view is a View instance
    }
    
    fun demonstrateContextUsage() {
        // Use Application Context for:
        // - Singleton pattern
        // - Background tasks
        // - Long-lived objects
        
        // Use Activity Context for:
        // - UI operations
        // - Fragments
        // - Short-lived operations
    }
}
```

## Configuration Changes

Handling configuration changes like screen rotation.

### Retaining Data

```kotlin
class MainActivity : AppCompatActivity() {
    
    private var savedState: String? = null
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Restore state
        savedState = savedInstanceState?.getString("state")
        
        setContentView(R.layout.activity_main)
    }
    
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        // Save state
        outState.putString("state", savedState)
    }
    
    override fun onRestoreInstanceState(savedInstanceState: Bundle) {
        super.onRestoreInstanceState(savedInstanceState)
        // Restore state
        savedState = savedInstanceState.getString("state")
    }
}
```

### Using ViewModel

```kotlin
class MainViewModel : ViewModel() {
    
    private val _uiState = MutableStateFlow<UiState>(UiState.Idle)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    // ViewModel survives configuration changes
    fun loadData() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            // Load data
            _uiState.value = UiState.Success(data)
        }
    }
}
```

## Activity Lifecycle

**What is the Activity Lifecycle?**

The Activity lifecycle is like the different stages of a person's day - from waking up to going to sleep. It represents all the different states an Activity can be in and the methods that get called when transitioning between these states.

**Why Understanding Lifecycle Matters:**

- **Resource Management**: Know when to start/stop expensive operations
- **Memory Efficiency**: Release resources when not needed
- **User Experience**: Provide smooth transitions between states
- **Data Persistence**: Save/restore data at the right times

**The Lifecycle States:**

Think of an Activity like a window in a house:
1. **Created** - Window is built but not visible
2. **Started** - Window is visible but might be behind another window
3. **Resumed** - Window is visible and user can interact with it
4. **Paused** - Window is partially hidden (like a dialog on top)
5. **Stopped** - Window is completely hidden
6. **Destroyed** - Window is torn down

**Lifecycle Methods Explained:**

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Called when activity is created
        // - Initialize activity
        // - Set content view
        // - Setup initial state
        // - Bind data to views
    }
    
    override fun onStart() {
        super.onStart()
        // Called when activity becomes visible
        // - Start animations
        // - Resume UI updates
        // - Register broadcast receivers
    }
    
    override fun onResume() {
        super.onResume()
        // Called when activity is in foreground
        // - Start GPS/Location
        // - Resume video/audio
        // - Start sensors
        // - User can now interact
    }
    
    override fun onPause() {
        super.onPause()
        // Called when activity is partially hidden
        // - Pause animations
        // - Release camera/sensors
        // - Save critical data
        // - Another activity is visible
    }
    
    override fun onStop() {
        super.onStop()
        // Called when activity is completely hidden
        // - Stop background work
        // - Release resources
        // - Activity no longer visible
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // Called when activity is being destroyed
        // - Release resources
        // - Unregister listeners
        // - Final cleanup
    }
    
    override fun onRestart() {
        super.onRestart()
        // Called when activity restarts from stopped
        // - Re-initialize components if needed
        // - Activity was stopped, now starting again
    }
}
```

**Key Points:**
- `onCreate()`: One-time setup, called only once
- `onStart()`: Activity becomes visible
- `onResume()`: User can interact with activity
- `onPause()`: Another activity is visible
- `onStop()`: Activity completely hidden
- `onDestroy()`: Final cleanup
- `onRestart()`: Returning from stopped state

## Fragment Lifecycle

**What is the Fragment Lifecycle?**

Fragments have a more complex lifecycle than Activities because they're tied to their host Activity. Think of a Fragment like a room in a house - it has its own lifecycle but is affected by what happens to the house.

**Why Fragment Lifecycle Matters:**

- **View Management**: Fragments can have views that are destroyed separately
- **State Persistence**: Need to save/restore Fragment state
- **Lifecycle Awareness**: Fragments must respond to Activity lifecycle changes
- **Memory Management**: Properly manage Fragment resources

**Fragment Lifecycle Methods:**

```kotlin
class MyFragment : Fragment() {
    
    override fun onAttach(context: Context) {
        super.onAttach(context)
        // Fragment attached to activity
        // - Get references to activity
        // - Initialize Fragment-specific components
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Fragment creation
        // - Initialize non-UI components
        // - Restore saved state
        // - No views available yet
    }
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate layout
        // - Create Fragment's view hierarchy
        // - Return root view
        return inflater.inflate(R.layout.fragment_main, container, false)
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // View fully created, safe to access views
        // - Setup view listeners
        // - Bind data to views
        // - Initialize UI components
    }
    
    override fun onStart() {
        super.onStart()
        // Fragment visible
        // - Start animations
        // - Resume UI updates
    }
    
    override fun onResume() {
        super.onResume()
        // Fragment active
        // - Start sensors
        // - Resume video/audio
    }
    
    override fun onPause() {
        super.onPause()
        // Fragment paused
        // - Pause animations
        // - Release sensors
    }
    
    override fun onStop() {
        super.onStop()
        // Fragment stopped
        // - Stop background work
        // - Release resources
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        // View destroyed, but fragment still exists
        // - Clean up view references
        // - Stop view-related operations
        // - Fragment can be recreated later
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // Fragment destroyed
        // - Final cleanup
        // - Release all resources
    }
    
    override fun onDetach() {
        super.onDetach()
        // Fragment detached from activity
        // - Clean up activity references
        // - Final cleanup
    }
}
```

**Key Differences from Activity:**
- `onAttach()`: Fragment connects to Activity
- `onCreateView()`: Creates Fragment's views
- `onViewCreated()`: Views are ready for use
- `onDestroyView()`: Views destroyed but Fragment lives on
- `onDetach()`: Fragment disconnects from Activity

## Application Lifecycle

**What is the Application Lifecycle?**

The Application lifecycle represents the entire app's journey from startup to shutdown. Think of it as the building that houses all your Activities and Fragments.

**Why Application Lifecycle Matters:**

- **Global State**: Manage app-wide resources and state
- **Memory Management**: Handle system memory pressure
- **Configuration Changes**: Respond to device changes
- **Initialization**: Setup app-wide components

**Application Lifecycle Methods:**

```kotlin
class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        // App created
        // - Initialize app-wide components
        // - Setup dependency injection
        // - Initialize analytics
        // - Setup crash reporting
        initializeApp()
    }
    
    private fun initializeApp() {
        // Setup app-wide components
        // - Initialize database
        // - Setup networking
        // - Initialize third-party libraries
    }
    
    override fun onLowMemory() {
        super.onLowMemory()
        // System is low on memory
        // - Release non-critical resources
        // - Clear caches
        // - Stop background operations
    }
    
    override fun onTrimMemory(level: Int) {
        super.onTrimMemory(level)
        // Memory trimming requested
        when (level) {
            TRIM_MEMORY_RUNNING_CRITICAL -> {
                // Release non-critical resources
                // - Clear image caches
                // - Stop background services
            }
            TRIM_MEMORY_UI_HIDDEN -> {
                // Release UI resources
                // - Clear view caches
                // - Release bitmap memory
            }
        }
    }
    
    override fun onConfigurationChanged(newConfig: Configuration) {
        super.onConfigurationChanged(newConfig)
        // Configuration changed
        // - Handle orientation changes
        // - Update language settings
        // - Adjust to new screen size
    }
}
```

**Key Points:**
- `onCreate()`: App-wide initialization
- `onLowMemory()`: System is running out of memory
- `onTrimMemory()`: System requests memory cleanup
- `onConfigurationChanged()`: Device configuration changed

## Intents and Intent Filters

**What are Intents?**

Intents are like messages that tell Android what you want to do. Think of them as sending a letter with instructions to another part of your app or even to other apps.

**Why Intents Matter:**

- **Component Communication**: Activities, Services, and Broadcast Receivers communicate
- **App Integration**: Your app can work with other apps
- **User Experience**: Seamless navigation between screens
- **System Integration**: Use system features like camera, contacts

**Types of Intents:**

1. **Explicit Intents** - "Send this to a specific address"
2. **Implicit Intents** - "Send this to anyone who can handle it"

**Intent Examples:**

```kotlin
// Explicit Intent - Start specific activity
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("key", "value")
startActivity(intent)

// Implicit Intent - Let system choose app
val intent = Intent(Intent.ACTION_VIEW)
intent.data = Uri.parse("https://www.example.com")
startActivity(intent)

// Intent with result - Get data back
val intent = Intent(this, SelectionActivity::class.java)
startActivityForResult(intent, REQUEST_CODE)
```

**Common Intent Actions:**

```kotlin
// View URL
Intent(Intent.ACTION_VIEW, Uri.parse("https://example.com"))

// Send text
Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "Message")
}

// Call phone
Intent(Intent.ACTION_DIAL, Uri.parse("tel:1234567890"))

// Send email
Intent(Intent.ACTION_SENDTO, Uri.parse("mailto:test@example.com"))

// Open camera
Intent(MediaStore.ACTION_IMAGE_CAPTURE)
```

**Intent Filters:**

Intent filters tell the system what your app can handle:

```xml
<activity android:name=".ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

## Permissions

**What are Android Permissions?**

Permissions are like asking for permission before entering someone's house. Android uses permissions to protect user privacy and system security.

**Why Permissions Matter:**

- **Privacy Protection**: Control what apps can access
- **Security**: Prevent malicious apps from harming the system
- **User Control**: Users decide what to allow
- **Transparency**: Users know what apps are doing

**Types of Permissions:**

1. **Normal Permissions** - Automatically granted
2. **Dangerous Permissions** - Require user approval
3. **Special Permissions** - Require special handling

**Permission Examples:**

```kotlin
// Normal permissions (auto-granted)
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

// Dangerous permissions (require runtime approval)
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

**Requesting Permissions:**

```kotlin
class MainActivity : AppCompatActivity() {
    
    private val permissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        if (isGranted) {
            // Permission granted
            openCamera()
        } else {
            // Permission denied
            showExplanationDialog()
        }
    }
    
    fun requestCameraPermission() {
        when {
            ContextCompat.checkSelfPermission(
                this,
                Manifest.permission.CAMERA
            ) == PackageManager.PERMISSION_GRANTED -> {
                openCamera()
            }
            shouldShowRequestPermissionRationale(Manifest.permission.CAMERA) -> {
                showExplanationDialog()
            }
            else -> {
                permissionLauncher.launch(Manifest.permission.CAMERA)
            }
        }
    }
}
```

## Context Types

**What is Context?**

Context is like the environment your app runs in. It provides access to app resources, system services, and other app components.

**Why Understanding Context Matters:**

- **Resource Access**: Get access to app resources
- **System Services**: Use system services like location, camera
- **Component Communication**: Start activities, services
- **Memory Management**: Choose right context to prevent leaks

**Types of Context:**

1. **Application Context** - Lives for entire app lifetime
2. **Activity Context** - Tied to Activity lifecycle
3. **View Context** - Tied to View's lifetime

**Context Usage:**

```kotlin
class MainActivity : AppCompatActivity() {
    
    // Application Context - Use for app-wide operations
    private val appContext: Context = applicationContext
    
    // Activity Context - Use for UI operations
    private val activityContext: Context = this
    
    fun demonstrateContextUsage() {
        // Use Application Context for:
        // - Singleton pattern
        // - Background tasks
        // - Long-lived objects
        
        // Use Activity Context for:
        // - UI operations
        // - Fragments
        // - Short-lived operations
    }
}
```

**When to Use Which:**

- **Application Context**: Singletons, global services, database, cache
- **Activity Context**: UI operations, starting activities, showing dialogs
- **View Context**: View-specific operations

## Configuration Changes

**What are Configuration Changes?**

Configuration changes happen when the device configuration changes, like rotating the screen or changing the language. Think of it like rearranging furniture in a room.

**Why Handle Configuration Changes:**

- **User Experience**: Maintain app state during changes
- **Resource Management**: Properly handle resource changes
- **Data Persistence**: Don't lose user data
- **Performance**: Avoid unnecessary recreation

**Handling Configuration Changes:**

```kotlin
class MainActivity : AppCompatActivity() {
    
    private var savedState: String? = null
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Restore state
        savedState = savedInstanceState?.getString("state")
        
        setContentView(R.layout.activity_main)
    }
    
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        // Save state
        outState.putString("state", savedState)
    }
    
    override fun onRestoreInstanceState(savedInstanceState: Bundle) {
        super.onRestoreInstanceState(savedInstanceState)
        // Restore state
        savedState = savedInstanceState.getString("state")
    }
}
```

**Using ViewModel for Configuration Changes:**

```kotlin
class MainViewModel : ViewModel() {
    
    private val _uiState = MutableStateFlow<UiState>(UiState.Idle)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    // ViewModel survives configuration changes
    fun loadData() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            // Load data
            _uiState.value = UiState.Success(data)
        }
    }
}
```

**Key Points:**
- `onSaveInstanceState()`: Save state before destruction
- `onRestoreInstanceState()`: Restore state after recreation
- ViewModel: Survives configuration changes automatically
- Bundle: Store simple data types

## Interview Questions & Answers

### Q1: Explain the Activity Lifecycle and when each method is called.

**Answer:**

The Activity lifecycle consists of seven callback methods:

**Lifecycle Flow:**

```kotlin
class LifecycleDemo : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Called when activity is created
        // - Initialize components
        // - Set up listeners
        // - Bind data
    }
    
    override fun onStart() {
        super.onStart()
        // Called when activity becomes visible
        // - Start animations
        // - Resume UI updates
        // - Register broadcast receivers
    }
    
    override fun onResume() {
        super.onResume()
        // Called when activity is in foreground
        // - Start GPS/Location
        // - Resume video/audio
        // - Start sensors
    }
    
    override fun onPause() {
        super.onPause()
        // Called when activity is partially hidden
        // - Pause animations
        // - Release camera/sensors
        // - Save critical data
    }
    
    override fun onStop() {
        super.onStop()
        // Called when activity is completely hidden
        // - Stop background work
        // - Release resources
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // Called when activity is being destroyed
        // - Release resources
        // - Unregister listeners
    }
    
    override fun onRestart() {
        super.onRestart()
        // Called when activity restarts from stopped
        // - Re-initialize components if needed
    }
}
```

**Key Points:**
- `onCreate()`: One-time initialization
- `onStart()`: Visibility begins
- `onResume()`: User can interact
- `onPause()`: Another activity visible
- `onStop()`: Activity completely hidden
- `onDestroy()`: Final cleanup
- `onRestart()`: Returning from stopped state

### Q2: What is the difference between Activity and Fragment lifecycles?

**Answer:**

**Key Differences:**

1. **Lifespan:**
   - Activity exists independently
   - Fragment is tied to hosting activity

2. **Initialization:**
   - Activity has `onCreate()`
   - Fragment has `onAttach()`, `onCreate()`, `onCreateView()`, `onViewCreated()`

3. **View Management:**
   - Activity has one content view
   - Fragment can have views that are destroyed separately

```kotlin
// Activity lifecycle
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main) // Views created here
}

// Fragment lifecycle
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    // No view here
}

override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    // Views created here
    return inflater.inflate(R.layout.fragment_main, container, false)
}

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    // Views fully created, safe to access
}

override fun onDestroyView() {
    super.onDestroyView()
    // Views destroyed but fragment still exists
}
```

**Key Points:**
- Fragment has more granular lifecycle
- Fragment views can be destroyed without destroying fragment
- Activity lifecycle affects fragment lifecycle
- Fragment needs container (activity or parent fragment)

### Q3: How do you handle configuration changes in Android?

**Answer:**

**Multiple strategies for handling configuration changes:**

**1. Using ViewModel (Recommended)**
```kotlin
class MainViewModel : ViewModel() {
    private val _data = MutableStateFlow<List<Item>>(emptyList())
    val data: StateFlow<List<Item>> = _data.asStateFlow()
    
    // ViewModel survives configuration changes
    fun loadData() {
        viewModelScope.launch {
            _data.value = repository.getData()
        }
    }
}

class MainActivity : AppCompatActivity() {
    private val viewModel: MainViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Observe ViewModel state
        lifecycleScope.launch {
            viewModel.data.collect { items ->
                updateUI(items)
            }
        }
    }
}
```

**2. Using savedInstanceState**
```kotlin
class MainActivity : AppCompatActivity() {
    private var savedData: String? = null
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Restore saved state
        savedData = savedInstanceState?.getString("data")
        
        setContentView(R.layout.activity_main)
    }
    
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        // Save state
        outState.putString("data", savedData)
    }
}
```

**3. Prevent Configuration Change**
```xml
<activity
    android:name=".MainActivity"
    android:configChanges="orientation|screenSize"
    android:screenOrientation="portrait">
</activity>
```

### Q4: What is an Intent and what are the types of Intents?

**Answer:**

**Intent** is a messaging object used to request an action from another app component.

**Types of Intents:**

**1. Explicit Intents**
```kotlin
// Explicitly specify the component to start
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("username", "john")
startActivity(intent)
```

**2. Implicit Intents**
```kotlin
// Specify action and let system resolve component
val intent = Intent(Intent.ACTION_VIEW)
intent.data = Uri.parse("https://www.example.com")
startActivity(intent)
```

**Common Intent Actions:**
```kotlin
// View URL
Intent(Intent.ACTION_VIEW, Uri.parse("https://example.com"))

// Send text
Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "Message")
}

// Call
Intent(Intent.ACTION_DIAL, Uri.parse("tel:1234567890"))

// Send email
Intent(Intent.ACTION_SENDTO, Uri.parse("mailto:test@example.com"))

// Open camera
Intent(MediaStore.ACTION_IMAGE_CAPTURE)
```

### Q5: Explain Android permissions system.

**Answer:**

**Permission System:**

**1. Normal Permissions (Auto-granted)**
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

**2. Dangerous Permissions (Runtime)**
```kotlin
// Declare in manifest
<uses-permission android:name="android.permission.CAMERA" />

// Request at runtime
class MainActivity : AppCompatActivity() {
    
    private val permissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        if (isGranted) {
            // Permission granted
            openCamera()
        } else {
            // Show explanation dialog
            if (shouldShowRequestPermissionRationale(Manifest.permission.CAMERA)) {
                showExplanationDialog()
            }
        }
    }
    
    fun requestCameraPermission() {
        when {
            ContextCompat.checkSelfPermission(
                this,
                Manifest.permission.CAMERA
            ) == PackageManager.PERMISSION_GRANTED -> {
                openCamera()
            }
            shouldShowRequestPermissionRationale(Manifest.permission.CAMERA) -> {
                showExplanationDialog()
            }
            else -> {
                permissionLauncher.launch(Manifest.permission.CAMERA)
            }
        }
    }
}
```

**3. Special Permissions**
```kotlin
// For special permissions like SYSTEM_ALERT_WINDOW
val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION)
intent.data = Uri.parse("package:$packageName")
startActivity(intent)
```

### Q6: What are the different types of Context and when to use them?

**Answer:**

**Context Types:**

**1. Application Context**
```kotlin
// Lives for app's entire lifetime
val appContext = applicationContext

// Use for:
// - Singleton pattern
class MySingleton(context: Context) {
    init {
        val applicationContext = context.applicationContext
        // Use applicationContext for long-lived objects
    }
}

// - Background tasks
class MyService : Service() {
    override fun onCreate() {
        super.onCreate()
        // Use applicationContext for DB operations
    }
}
```

**2. Activity Context**
```kotlin
// Tied to activity lifecycle
class MainActivity : AppCompatActivity() {
    private val activityContext: Context = this
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Use activity context for UI operations
        val intent = Intent(this, SecondActivity::class.java)
        startActivity(intent)
        
        // Show Toast
        Toast.makeText(this, "Message", Toast.LENGTH_SHORT).show()
    }
}
```

**3. View Context**
```kotlin
// Tied to view's lifetime
class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // Use view context for view-specific operations
        val context = view.context
    }
}
```

**When to Use Which:**

- **Application Context**: Singletons, global services, database, cache
- **Activity Context**: UI operations, starting activities, showing dialogs
- **View Context**: View-specific operations

**Common Mistake:**
```kotlin
// WRONG: Using activity context for long-lived object
class BadExample {
    private val context: Context // Activity context - will leak!
}

// CORRECT: Use application context
class GoodExample {
    private val context: Context = context.applicationContext
}
```

### Q7: What is the difference between onResume() and onStart()?

**Answer:**

**onStart()**:
- Called when activity becomes **visible**
- Another activity might be on top (e.g., dialog)
- Good for: UI preparation, animations

**onResume()**:
- Called when activity is **in foreground**
- User can interact with activity
- Good for: Starting sensors, GPS, cameras

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onStart() {
        super.onStart()
        // Activity is visible but user might not interact yet
        // Example: Dialog overlay on activity
        initializeUI()
    }
    
    override fun onResume() {
        super.onResume()
        // Activity is in foreground, user can interact
        startGPS()
        startCamera()
        resumeAnimations()
    }
    
    override fun onPause() {
        super.onPause()
        // Activity partially hidden or lost focus
        pauseCamera()
        pauseGPS()
    }
    
    override fun onStop() {
        super.onStop()
        // Activity no longer visible
        stopUIUpdates()
    }
}
```

**Key Points:**
- Multiple `onResume()` calls between one `onStart()`
- `onResume()` indicates user interaction is possible
- `onStart()` indicates visibility only

---

## Navigation

**Previous:** [Introduction](introduction.md)

**Next:** [Kotlin Programming](kotlin-programming.md)

