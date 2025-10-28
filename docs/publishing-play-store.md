---
layout: default
title: "Publishing to Play Store - Interview Preparation"
---

# Publishing to Play Store

## Table of Contents
- [Account Types](#account-types)
- [Publishing Process](#publishing-process)
- [Release Tracks](#release-tracks)
- [Interview Questions & Answers](#interview-questions--answers)

## Account Types

### Personal vs Organization Accounts

| Feature | Personal Account | Organization Account |
|---------|-----------------|---------------------|
| Registration Fee | $25 (one-time) | $25 (one-time) |
| Company Name Display | No | Yes |
| Bulk Management | No | Yes |
| Team Management | Limited | Advanced |
| Payment Profiles | Single | Multiple |
| Analytics | Basic | Advanced |
| App Management | Manual | Bulk APIs |

## Interview Questions & Answers

### Q1: Compare Personal vs Organization Play Store accounts

**Answer:**

**Personal Account:**
- Individual developers
- Simple setup
- Limited features
- Suitable for personal projects

**Organization Account:**
- Companies and teams
- Advanced team management
- Bulk app operations
- Better for commercial apps

### Q2: What is the complete publishing process to Play Store?

**Answer:**

**Step-by-Step Process:**

**1. Prepare Your App**
```gradle
// build.gradle
android {
    compileSdk 34
    
    defaultConfig {
        applicationId "com.example.myapp"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"
    }
    
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

**2. Generate Signed Bundle**
```bash
# Generate AAB file
./gradlew bundleRelease

# Output: app/build/outputs/bundle/release/app-release.aab
```

**3. Create Play Console Account**
- Register at https://play.google.com/console
- Pay $25 one-time fee
- Complete developer account verification

**4. Create App Listing**
- App name, description, screenshots
- Feature graphic (1024x500)
- Privacy policy URL
- App category

**5. Upload AAB**
```bash
# Upload through Play Console or CLI
fastlane upload_bundle
```

**6. Fill App Content Rating**
- Complete questionnaire
- Get age rating

**7. Set Pricing & Distribution**
- Free or paid app
- Countries distribution

**8. Review & Submit**
- Check all sections
- Submit for review
- Wait for approval (1-7 days)

### Q3: Explain release tracks (internal, alpha, beta, production)

**Answer:**

**Internal Testing:**
```yaml
Purpose: Rapid testing with trusted testers
Audience: Up to 100 users
Approval: Quick (usually minutes)
Use case: Development testing
```

**Closed Alpha Testing:**
```yaml
Purpose: Limited external testing
Audience: Up to 500 users
Approval: Requires review
Use case: Feature validation
```

**Closed Beta Testing:**
```yaml
Purpose: Broader testing
Audience: Up to 20,000 users
Approval: Requires review
Use case: Pre-release testing
```

**Open Beta:**
```yaml
Purpose: Public beta
Audience: Unlimited
Approval: Requires review
Use case: Public testing
```

**Production:**
```yaml
Purpose: Public release
Audience: All users
Approval: Full review (1-7 days)
Use case: Stable release
```

**Rollout Strategy:**
```
Internal → Alpha → Beta → Production (20%) → Production (100%)
```

### Q4: What is the difference between AAB and APK?

**Answer:**

**APK (Android Package):**
```kotlin
// Traditional format
// Pros: Direct installation, universal
// Cons: Larger size, no optimization
```

**AAB (Android App Bundle):**
```kotlin
// New format (required for Play Store)
// Pros: 
//   - Smaller downloads (30% size reduction)
//   - Dynamic delivery
//   - Split per architecture/device
//   - Download only what's needed
// Cons: Can't install directly (must go through Play Store)
```

**Size Comparison:**
```yaml
APK: 50 MB
AAB: 35 MB (after compression)
Download size: ~20 MB (device-specific)
```

**Dynamic Delivery:**
```gradle
android {
    bundle {
        language {
            enableSplit = true
        }
        density {
            enableSplit = true
        }
        abi {
            enableSplit = true
        }
    }
}
```

### Q5: How to sign an Android app?

**Answer:**

**Generate Keystore:**
```bash
keytool -genkeypair -v -storetype PKCS12 \
  -keystore my-release-key.keystore \
  -alias my-key-alias \
  -keyalg RSA -keysize 2048 \
  -validity 10000
```

**Sign with Gradle:**
```gradle
// build.gradle
android {
    signingConfigs {
        release {
            storeFile file('my-release-key.keystore')
            storePassword 'password'
            keyAlias 'my-key-alias'
            keyPassword 'password'
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

**Automated Signing (Recommended):**
```kotlin
// Use Play App Signing in Play Console
// Google manages your signing key
// Lost key? Google can re-sign updates
```

### Q6: Explain staged rollout strategy

**Answer:**

**Staged Rollout Benefits:**
- Reduce risk of bugs affecting all users
- Monitor crash reports and analytics
- Gradual release to catch issues early

**Rollout Phases:**
```yaml
Phase 1: 5% of users (Day 1-2)
  - Monitor crash rate
  - Check analytics
  - Review ratings
  
Phase 2: 20% of users (Day 3-5)
  - Check feedback
  - Monitor issues
  - Adjust if needed
  
Phase 3: 50% of users (Day 6-8)
  - Continue monitoring
  - Review metrics
  
Phase 4: 100% (Day 9+)
  - Full rollout
  - Complete release
```

**Implementation:**
```kotlin
// In Play Console
// Production → Release → Manage release → Staged rollout
// Set percentage and release
```

### Q7: What are common app rejection reasons?

**Answer:**

**Top Rejection Reasons:**

1. **Privacy Policy Missing**
   ```yaml
   Issue: No privacy policy URL
   Fix: Add privacy policy accessible via web
   ```

2. **Inappropriate Content**
   ```yaml
   Issue: Violates content policy
   Fix: Review content guidelines
   ```

3. **Permissions Misuse**
   ```kotlin
   // BAD
   <uses-permission android:name="android.permission.CAMERA"/>
   // Using camera permission but camera feature not evident
   
   // GOOD
   // Clearly show camera usage in app description/screenshots
   ```

4. **Incomplete App Information**
   ```yaml
   Issue: Missing screenshots, description
   Fix: Complete all app listing sections
   ```

5. **Performance Issues**
   ```kotlin
   // Crashes on startup
   // Fix: Test on multiple devices
   // Use Firebase Crashlytics
   ```

6. **Security Vulnerabilities**
   ```kotlin
   // Using insecure http instead of https
   // Fix: Use https for all network calls
   ```

7. **Misleading Functionality**
   ```yaml
   Issue: App doesn't do what description says
   Fix: Ensure app matches description
   ```

### Q8: How to set up in-app products?

**Answer:**

**In-App Products Types:**

**1. Managed Products (One-time purchase)**
```kotlin
// In Google Play Console
// Monetize → Products → In-app products → Create product

class PurchaseManager(private val billingClient: BillingClient) {
    
    fun purchaseProduct(activity: Activity, productId: String) {
        val params = BillingFlowParams.newBuilder()
            .setSkuDetails(skuDetails)
            .build()
        billingClient.launchBillingFlow(activity, params)
    }
    
    fun handlePurchase(purchase: Purchase) {
        if (purchase.purchaseState == PurchaseState.PURCHASED) {
            // Grant product to user
            grantProduct(purchase.products.first())
        }
    }
}
```

**2. Subscriptions**
```kotlin
class SubscriptionManager {
    fun checkSubscriptionStatus(callback: (Boolean) -> Unit) {
        // Check if user has active subscription
        billingClient.queryPurchasesAsync(
            QueryPurchasesParams.newBuilder()
                .setProductType(BillingClient.SkuType.SUBS)
                .build()
        ) { result ->
            callback(result.purchasesList.isNotEmpty())
        }
    }
}
```

**3. Consumable Products**
```kotlin
class ConsumableManager {
    suspend fun consumePurchase(purchase: Purchase) {
        val consumeParams = ConsumeParams.newBuilder()
            .setPurchaseToken(purchase.purchaseToken)
            .build()
        
        billingClient.consumeAsync(consumeParams) { result ->
            if (result.responseCode == BillingClient.BillingResponseCode.OK) {
                // Purchase consumed
            }
        }
    }
}
```

### Q9: Explain app versioning and updates

**Answer:**

**Versioning Strategy:**
```gradle
android {
    defaultConfig {
        versionCode 1  // Integer, must be incremented
        versionName "1.0" // String, user-friendly
    }
    
    // Increment versionCode for each release
    // Update versionName for user-visible version
}
```

**Version Increment:**
```kotlin
// Major version (API breaking changes)
versionCode 100
versionName "2.0.0"

// Minor version (new features)
versionCode 51
versionName "1.5.0"

// Patch version (bug fixes)
versionCode 3
versionName "1.0.1"
```

**Update Strategy:**

**Forced Updates:**
```kotlin
// Check version on app start
class VersionChecker {
    fun checkForUpdates(currentVersion: Int, remoteVersion: Int) {
        if (currentVersion < remoteVersion) {
            showUpdateDialog(forced = true)
        }
    }
}
```

**Optional Updates:**
```kotlin
// Let user choose
fun showUpdateDialog(forced: Boolean) {
    if (forced) {
        // Disable dismiss, must update
    } else {
        // Allow later option
    }
}
```

**Update Best Practices:**
- Test updates on staging track first
- Monitor crash reports after updates
- Use staged rollouts
- Provide release notes
- Support multiple versions if needed

---

**Previous:** [Gradle & Maven](gradle-maven.md)

**Next:** [Security](security.md)

