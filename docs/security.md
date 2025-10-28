---
layout: default
title: "Security - Interview Preparation"
---

# Security

## Table of Contents
- [SSL Pinning](#ssl-pinning)
- [Data Encryption](#data-encryption)
- [API Key Security](#api-key-security)
- [Interview Questions & Answers](#interview-questions--answers)

## SSL Pinning

```kotlin
val certificatePinner = CertificatePinner.Builder()
    .add("api.example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .build()

val okHttpClient = OkHttpClient.Builder()
    .certificatePinner(certificatePinner)
    .build()
```

## Interview Questions & Answers

### Q1: How to implement SSL Pinning?

**Answer:**

SSL Pinning prevents man-in-the-middle attacks by verifying server certificates.

```kotlin
class PinnedOkHttpClient {
    fun getClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .certificatePinner(
                CertificatePinner.Builder()
                    .add("api.example.com", "sha256/+HrF5hK7ff/QLHP2nm6BHB/b3b5bFBF4+NV6XYQ=")
                    .build()
            )
            .build()
    }
}
```

### Q2: How to secure sensitive data in Android app?

**Answer:**

**Conceptual Understanding:**

Securing sensitive data is crucial to protect user privacy and comply with regulations. Data can be encrypted at rest (stored encrypted) or in transit (encrypted during transmission).

**Security Layers:**
1. Encryption: Transform data into unreadable format
2. Key management: Securely store encryption keys
3. Hashing: One-way transformation for passwords
4. Obfuscation: Make reverse engineering harder

**Common Sensitive Data:**
- User credentials
- API keys
- Personal information
- Payment data
- Session tokens

**Encryption:**
```kotlin
class SecureStorage {
    private val cipher = Cipher.getInstance("AES/GCM/NoPadding")
    
    fun encryptData(data: String, key: SecretKey): String {
        cipher.init(Cipher.ENCRYPT_MODE, key)
        val encrypted = cipher.doFinal(data.toByteArray())
        return Base64.encodeToString(encrypted, Base64.NO_WRAP)
    }
}
```

### Q3: How to secure API keys?

**Answer:**

**Conceptual Understanding:**

API keys are credentials that authenticate your app to backend services. Exposing API keys in your code is a security risk as anyone can reverse engineer your APK and steal them.

**Security Approaches:**
1. Build-time injection (hidden in APK but still extractable)
2. Native code (more difficult to extract)
3. Server-side proxy (keys never in app)
4. Obfuscation (makes extraction harder)

**Important Note:**
No method is 100% secure. The best practice is to use server-side validation and limit what API keys can do.

**Option 1: Build Config**
```kotlin
// build.gradle
android {
    buildTypes.each {
        it.buildConfigField "String", "API_KEY", "\"your_key_here\""
    }
}

// Usage
val apiKey = BuildConfig.API_KEY
```

**Option 2: NDK/Native**
```kotlin
external fun getNativeApiKey(): String
```

### Q4: How to prevent reverse engineering?

**Answer:**

**Conceptual Understanding:**

While you cannot completely prevent reverse engineering, you can make it significantly more difficult. The goal is to protect intellectual property and make attacks more expensive than the reward.

**Attack Vectors:**
- APK decompilation (dex2jar, JADX)
- Binary analysis
- Runtime manipulation (Frida, Xposed)
- Network traffic interception

**Protection Layers:**
1. Code obfuscation (rename classes/methods)
2. String encryption
3. Control flow obfuscation
4. Anti-debugging checks
5. Root detection

**1. ProGuard/R8:**
```gradle
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt')
        }
    }
}
```

**2. Code Obfuscation:**
```kotlin
// Original
fun authenticateUser(username: String, password: String): Boolean { }

// Obfuscated
fun a(b: String, c: String): Boolean { }
```

### Q5: How to use ProGuard/R8 effectively?

**Answer:**

**Conceptual Understanding:**

ProGuard/R8 shrinks, optimizes, and obfuscates your code. However, it needs guidance on what to keep and what to remove, especially for reflection-based code.

**Key Concepts:**
- **Shrinking**: Removes unused code
- **Obfuscation**: Renames classes/methods to make reverse engineering harder
- **Optimization**: Improves bytecode performance
- **Keep Rules**: Prevent removal of code needed at runtime

**Common Pitfalls:**
- Serialization classes removed
- Reflection-based code broken
- Native method names changed
- Dynamic class loading fails

**Best Practices:**
- Test thoroughly with R8 enabled
- Keep all models and DTOs
- Keep annotation classes
- Test all app functionality after obfuscation

```proguard
# ProGuard rules

# Keep data classes
-keep class com.example.model.** { *; }

# Keep serialization
-keepattributes Signature
-keepattributes *Annotation*

# Remove logging in release
-assumenosideeffects class android.util.Log {
    public static *** d(...);
    public static *** v(...);
}
```

### Q6: How to implement encryption at rest?

**Answer:**

**Conceptual Foundation:**

Encryption at rest means encrypting data before storing it persistently (database, files, SharedPreferences). This protects data even if the device is compromised or the storage medium is accessed directly.

**Encryption Algorithms:**
- AES (Advanced Encryption Standard): Industry standard
- GCM mode: Provides authentication in addition to encryption
- 256-bit keys: Strong security

**Key Management:**
- Store keys in Android Keystore (hardware-backed if available)
- Never hardcode encryption keys
- Use different keys for different data types
- Rotate keys periodically

**Best Practices:**
- Use Android Keystore for key storage
- Use authenticated encryption (AES-GCM)
- Protect keys with biometric/PIN
- No fallback to unencrypted storage

```kotlin
class EncryptedStorage(private val secretKey: SecretKey) {
    private val transformation = "AES/GCM/NoPadding"
    
    fun encrypt(data: String): String {
        val cipher = Cipher.getInstance(transformation)
        cipher.init(Cipher.ENCRYPT_MODE, secretKey)
        
        val encrypted = cipher.doFinal(data.toByteArray(Charsets.UTF_8))
        return Base64.encodeToString(encrypted, Base64.NO_WRAP)
    }
    
    fun decrypt(encryptedData: String): String {
        val data = Base64.decode(encryptedData, Base64.NO_WRAP)
        val cipher = Cipher.getInstance(transformation)
        cipher.init(Cipher.DECRYPT_MODE, secretKey)
        
        val decrypted = cipher.doFinal(data)
        return String(decrypted, Charsets.UTF_8)
    }
}
```

### Q7: Explain certificate pinning implementation

**Answer:**

**Conceptual Foundation:**

Certificate pinning is a security mechanism that allows your app to verify that it's communicating with the legitimate server by "pinning" the server's public key in your app.

**How Certificate Pinning Works:**

Normally, HTTPS trusts any certificate validated by a certificate authority (CA). Certificate pinning bypasses the CA and directly verifies the server's certificate against a pre-configured list stored in your app.

**When to Use:**
- High-security applications
- Financial apps
- Healthcare apps
- Apps handling sensitive user data

**Trade-offs:**
- Pros: Prevents MITM attacks, more secure
- Cons: Require app update when server certificate changes, maintenance overhead

**Certificate Pinning Process:**

1. **Extract certificate pin:**
```bash
openssl s_client -connect api.example.com:443 | openssl x509 -pubkey -noout | openssl rsa -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
```

2. **Implement in code:**
```kotlin
class CertificatePinningSetup {
    fun getClient(): OkHttpClient {
        val certificatePinner = CertificatePinner.Builder()
            .add("api.example.com", "sha256/+HrF5hK7ff/QLHP2nm6BHB/b3b5bFBF4+NV6XYQ=")
            .add("api.example.com", "sha256/second_pin")
            .build()
        
        return OkHttpClient.Builder()
            .certificatePinner(certificatePinner)
            .build()
    }
}
```

**Why Certificate Pinning:**
- Prevents MITM attacks
- Ensures server identity
- Increases app security

### Q8: How to implement secure communication?

**Answer:**

Always enforce HTTPS with modern TLS, validate certificates, and pin when needed.
Harden network stacks by disabling insecure protocols/ciphers and handling SSL errors safely.

**Conceptual Understanding:**

Secure communication is essential for protecting user data in transit. HTTPS should be the default for all network communication in modern Android apps.

**Security Layers:**
1. **TLS/SSL**: Encryption and authentication
2. **Certificate Pinning**: Verify server identity
3. **Strong Cipher Suites**: Use modern encryption
4. **Certificate Validation**: Check certificate chains

**Best Practices:**
- Only use HTTPS (never HTTP)
- Validate server certificates
- Use certificate pinning for high-security apps
- Keep TLS/SSL libraries updated
- Handle SSL errors gracefully

**HTTPS Only:**
```kotlin
val client = OkHttpClient.Builder()
    .addInterceptor { chain ->
        val request = chain.request()
        if (request.url.scheme != "https") {
            throw IOException("HTTP not allowed")
        }
        chain.proceed(request)
    }
    .build()
```

**TLS Configuration:**
```kotlin
val sslContext = SSLContext.getInstance("TLS")
sslContext.init(null, trustManagers, SecureRandom())

val client = OkHttpClient.Builder()
    .sslSocketFactory(sslContext.socketFactory, trustManager)
    .build()
```

---

**Previous:** [Publishing to Play Store](publishing-play-store.md)

**Next:** [Advanced Topics](advanced-topics.md)

