---
layout: default
title: "Networking - Interview Preparation"
---

# Networking

## Table of Contents
- [Retrofit](#retrofit)
- [OkHttp](#okhttp)
- [Ktor](#ktor)
- [Interview Questions & Answers](#interview-questions--answers)

## Retrofit

```kotlin
interface ApiService {
    @GET("users")
    suspend fun getUsers(): List<User>
    
    @POST("users")
    suspend fun createUser(@Body user: User): User
}

class MainViewModel @Inject constructor(
    private val api: ApiService
) : ViewModel() {
    
    fun loadUsers() {
        viewModelScope.launch {
            try {
                val users = api.getUsers()
                _uiState.value = UiState.Success(users)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message)
            }
        }
    }
}
```

## Interview Questions & Answers

### Q1: Compare Retrofit, Volley, and Ktor

**Answer:**

**Retrofit:**
- Type-safe HTTP client
- Works with Kotlin Coroutines
- Most popular in Android
- Easy to use

**Volley:**
- Old library
- Not actively developed
- Not recommended for new projects

**Ktor:**
- Modern Kotlin-native
- Supports server and client
- Flexible architecture

### Q2: How to handle errors in Retrofit?

**Answer:**

**Conceptual Foundation:**

Error handling in network operations is critical for a good user experience. Network calls can fail for various reasons: no connection, server errors, timeouts, or unexpected data formats.

**Error Categories:**
- Network errors: No internet, timeouts
- HTTP errors: 4xx (client errors), 5xx (server errors)
- Parsing errors: Malformed JSON, unexpected data
- Authentication errors: 401 Unauthorized, token expired

**Best Practices:**
- Always use try-catch for network operations
- Provide meaningful error messages to users
- Log errors for debugging
- Handle different error types appropriately

```kotlin
class ApiRepository(private val api: ApiService) {
    suspend fun getUsers(): Result<List<User>> {
        return try {
            val users = api.getUsers()
            Result.success(users)
        } catch (e: HttpException) {
            Result.failure(Exception("HTTP error: ${e.code()}"))
        } catch (e: IOException) {
            Result.failure(Exception("Network error"))
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### Q3: Explain SSL Pinning implementation

**Answer:**

**Conceptual Understanding:**

SSL Pinning adds an extra layer of security beyond default certificate validation. Instead of trusting any certificate in the root certificate store, you pin your app to specific certificates.

**Why SSL Pinning:**
- Prevents Man-in-the-Middle (MITM) attacks
- Ensures you're communicating with the correct server
- Protects against compromised certificate authorities
- Industry best practice for sensitive data

**How It Works:**
Your app stores the server's certificate (or public key hash) and only accepts connections if the server's certificate matches the pinned one.

**Trade-offs:**
- More secure
- Requires certificate updates when server certificate changes
- Additional implementation complexity

```kotlin
val certificatePinner = CertificatePinner.Builder()
    .add("api.example.com", "sha256/+HrF5hK7ff/QLHP2nm6BHB/b3b5bFBF4+NV6XYQ=")
    .build()

val client = OkHttpClient.Builder()
    .certificatePinner(certificatePinner)
    .build()
```

### Q4: How to implement caching strategies?

**Answer:**

**Conceptual Foundation:**

Caching improves app performance by storing frequently accessed data locally, reducing network requests and providing faster responses.

**Caching Strategies:**
- **Cache-First**: Serve from cache, fallback to network if unavailable
- **Network-First**: Fetch from network, use cache as backup
- **Stale-While-Revalidate**: Show stale data immediately, update in background
- **Cache-Control**: HTTP headers to control cache behavior

**Benefits:**
- Faster responses
- Reduced data usage
- Works offline
- Lower server load

**Cache Considerations:**
- Cache size limits
- Cache invalidation
- Cache expiration
- Memory vs disk caching

```kotlin
val cache = Cache(context.cacheDir, 10 * 1024 * 1024)

val client = OkHttpClient.Builder()
    .cache(cache)
    .addInterceptor(CacheInterceptor())
    .build()

class CacheInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        val response = chain.proceed(request)
        return response.newBuilder()
            .header("Cache-Control", "public, max-age=3600")
            .build()
    }
}
```

### Q5: Explain OkHttp interceptors

**Answer:**

**Conceptual Understanding:**

Interceptors are a powerful feature of OkHttp that allow you to observe, modify, and short-circuit HTTP requests and responses. They form a chain where each interceptor can process the request before passing it to the next.

**Interceptor Types:**
- **Application Interceptors**: Run before network requests
- **Network Interceptors**: Run after redirects, before final network call

**Common Use Cases:**
- Logging requests/responses
- Adding authentication headers
- Modifying requests/responses
- Retrying failed requests
- Caching responses

**Processing Order:**
Request → Application Interceptors → Network → Network Interceptors → Response

```kotlin
// Logging interceptor
val loggingInterceptor = HttpLoggingInterceptor().apply {
    level = HttpLoggingInterceptor.Level.BODY
}

// Auth interceptor
class AuthInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer $token")
            .build()
        return chain.proceed(request)
    }
}

val client = OkHttpClient.Builder()
    .addInterceptor(loggingInterceptor)
    .addInterceptor(AuthInterceptor())
    .build()
```

### Q6: How to implement retry logic with exponential backoff?

**Answer:**

**Conceptual Understanding:**

Retry logic with exponential backoff is a strategy for handling transient network failures. Instead of immediate retries that could overwhelm a struggling server, you wait progressively longer between attempts.

**The Strategy:**
- First retry after 1 second
- Second retry after 2 seconds
- Third retry after 4 seconds
- etc.

**Why Exponential Backoff:**
- Avoids overwhelming servers that are temporarily overloaded
- Gives servers time to recover
- Reduces unnecessary network traffic
- Improves success rate for transient failures

**Factors to Consider:**
- Maximum retry attempts
- Base delay time
- Maximum delay cap
- Which errors to retry (not all errors should be retried)

```kotlin
class RetryInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        var request = chain.request()
        var response: Response? = null
        var exception: IOException? = null
        
        repeat(3) { attemptNumber ->
            try {
                response = chain.proceed(request)
                if (response.isSuccessful) return response
            } catch (e: IOException) {
                exception = e
            }
            
            val delay = (2.0.pow(attemptNumber)).toLong() * 1000
            Thread.sleep(delay)
        }
        
        throw exception ?: IOException("Request failed")
    }
}
```

### Q7: How to handle authentication tokens?

**Answer:**

**Conceptual Understanding:**

Authentication tokens (like JWT tokens) are credentials that prove a user's identity. They need to be securely stored and automatically included in API requests.

**Token Lifecycle:**
1. Obtain token after successful login
2. Store token securely (encrypted SharedPreferences or Keystore)
3. Attach to all API requests
4. Refresh when expired
5. Clear when user logs out

**Security Considerations:**
- Never expose tokens in URLs or logs
- Use HTTPS for token transmission
- Implement token refresh mechanism
- Handle token expiration gracefully

**Implementation Approaches:**
- Store in encrypted storage
- Add via OkHttp interceptor
- Refresh automatically when expired

```kotlin
class TokenManager @Inject constructor() {
    private var token: String? = null
    
    fun saveToken(token: String) {
        this.token = token
    }
    
    fun getToken(): String? = token
}

class AuthInterceptor(private val tokenManager: TokenManager) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        
        if (request.header("Authorization") == null) {
            tokenManager.getToken()?.let { token ->
                val newRequest = request.newBuilder()
                    .addHeader("Authorization", "Bearer $token")
                    .build()
                return chain.proceed(newRequest)
            }
        }
        
        return chain.proceed(request)
    }
}
```

### Q8: How to handle file uploads/downloads with progress?

**Answer:**

**Conceptual Understanding:**

Handling file uploads and downloads with progress feedback is crucial for a good user experience, especially when dealing with large files. Users need to see download/upload progress and know the operation is still working.

**Progress Tracking Approaches:**
- Measure bytes transferred vs total bytes
- Update UI on progress milestones
- Handle progress in background thread
- Provide cancel functionality

**Implementation Strategies:**
- Use OkHttp's request body wrapping for uploads
- Custom response body wrapper for downloads
- Calculate progress percentage
- Emit progress updates via Flow or callbacks

```kotlin
// Upload with progress
interface ApiService {
    @Multipart
    @POST("upload")
    suspend fun uploadFile(
        @Part file: MultipartBody.Part,
        @Header("Content-Length") contentLength: Long
    ): Response<UploadResponse>
}

// Download with progress
suspend fun downloadWithProgress(url: String, callbacks: ProgressCallbacks) {
    val request = Request.Builder().url(url).build()
    val client = OkHttpClient()
    val response = client.newCall(request).execute()
    
    val responseBody = response.body
    val totalBytes = responseBody?.contentLength() ?: 0
    
    responseBody?.source()?.apply {
        var totalBytesRead = 0L
        
        while (true) {
            val bytesRead = read(Buffer(), 8192)
            if (bytesRead == -1L) break
            totalBytesRead += bytesRead
            callbacks.onProgress(totalBytesRead, totalBytes)
        }
    }
}
```

---

**Previous:** [Data Persistence](data-persistence.md)

**Next:** [Dependency Injection](dependency-injection.md)

