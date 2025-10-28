---
layout: default
title: "Data Persistence - Interview Preparation"
---

# Data Persistence

## Table of Contents
- [Room Database](#room-database)
- [DataStore](#datastore)
- [SharedPreferences](#sharedpreferences)
- [Interview Questions & Answers](#interview-questions--answers)

## Room Database

```kotlin
@Entity
data class User(
    @PrimaryKey val uid: Int,
    val name: String,
    val email: String
)

@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    fun getAll(): Flow<List<User>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: User)
}

@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

## Interview Questions & Answers

### Q1: Compare Room, SQLite, and Realm

**Answer:**

**Room:**
- Abstraction over SQLite
- Compile-time queries validation
- Kotlin Coroutines support
- Official Android library

**SQLite:**
- Raw SQL
- More control
- Steeper learning curve

**Realm:**
- NoSQL database
- Easiest API
- Larger app size
- Not maintained as actively

### Q2: How to handle database migrations in Room?

**Answer:**

**Conceptual Understanding:**

Database migration is the process of upgrading your database schema when you change the structure of your database. Room provides a type-safe way to handle schema changes, ensuring that user data isn't lost when your app updates.

**Why Migrations Matter:**
- Preserve user data across app updates
- Handle schema changes safely
- Avoid app crashes from incompatible schemas

**The Migration Process:**
1. Increment the database version number
2. Define a Migration object specifying how to transform old schema to new
3. Add the migration to your database builder
4. Room automatically runs migrations when the database is opened

**Migration Strategies:**
- Add new columns
- Delete unused columns
- Rename tables
- Change column types

```kotlin
@Database(entities = [User::class], version = 2)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null
        
        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"
                )
                    .addMigrations(MIGRATION_1_2)
                    .build()
                INSTANCE = instance
                instance
            }
        }
        
        private val MIGRATION_1_2 = object : Migration(1, 2) {
            override fun migrate(database: SupportSQLiteDatabase) {
                database.execSQL("ALTER TABLE user ADD COLUMN age INTEGER NOT NULL DEFAULT 0")
            }
        }
    }
}
```

### Q3: Compare DataStore with SharedPreferences

**Answer:**

**Conceptual Background:**

DataStore is Google's modern replacement for SharedPreferences. It solves several issues with SharedPreferences while maintaining a similar API for simple key-value storage.

**Why DataStore Over SharedPreferences:**

SharedPreferences (Legacy):
- Synchronous API blocks the UI thread
- No type safety
- No error handling for corrupt data
- All-or-nothing commits
- No way to observe data changes

DataStore (Modern):
- Asynchronous, non-blocking API using Kotlin coroutines
- Type-safe with Kotlin's type system
- Built-in error handling
- Can handle partial failures
- Observable data using Flow

**When to Use:**
- DataStore: All new code, complex data, need Flow support
- SharedPreferences: Legacy code, simple use cases, quick migrations

**DataStore Advantages:**
- Type-safe APIs
- Coroutines/Flow support
- Asynchronous API
- Data consistency

**SharedPreferences:**
- Synchronous API
- Not type-safe
- No observable data

```kotlin
// DataStore
val dataStore: DataStore<Preferences> = context.createDataStore("settings")

suspend fun saveUsername(username: String) {
    dataStore.edit { preferences ->
        preferences[stringPreferencesKey("username")] = username
    }
}

// SharedPreferences (old way)
val prefs = context.getSharedPreferences("settings", Context.MODE_PRIVATE)
prefs.edit().putString("username", username).apply()
```

### Q4: How to implement offline-first architecture?

**Answer:**

**Conceptual Foundation:**

Offline-first architecture ensures your app works seamlessly even when there's no network connection. The key principle is to cache data locally and sync when online.

**Design Principles:**
1. Local cache as single source of truth
2. Network as synchronization mechanism
3. Always show cached data immediately
4. Update cache silently in background
5. Handle conflicts gracefully

**Benefits:**
- Instant UI updates (no waiting for network)
- Works in poor network conditions
- Better user experience
- Reduced server load
- Reliable in low-connectivity areas

**Implementation Strategy:**
```kotlin
class OfflineFirstRepository(
    private val api: ApiService,
    private val dao: UserDao
) {
    
    fun getUsers(): Flow<List<User>> = flow {
        // Emit cached data first
        emitAll(dao.getAll())
        
        try {
            // Try to fetch from network
            val remote = api.getUsers()
            dao.insertAll(remote)
            emit(remote)
        } catch (e: Exception) {
            // Return cached data if network fails
        }
    }
}
```

### Q5: How to optimize Room database performance?

**Answer:**

**Performance Optimization Philosophy:**

Room database performance depends on several factors: query complexity, data volume, indexing, and transaction management. Optimization is about minimizing database operations and maximizing efficiency.

**Key Optimization Areas:**
- Indexes for frequently queried columns
- Efficient queries (avoid SELECT *)
- Bulk operations instead of individual operations
- Pagination for large datasets
- Transactions for multiple related operations

**1. Use Indexes:**
```kotlin
@Entity(indices = [Index(value = ["email"], unique = true)])
data class User(
    @PrimaryKey val id: Int,
    val name: String,
    val email: String
)
```

**2. Use @Transaction:**
```kotlin
@Transaction
suspend fun updateUsers(users: List<User>) {
    deleteAll()
    insertAll(users)
}
```

**3. Pagination:**
```kotlin
@Query("SELECT * FROM user LIMIT :pageSize OFFSET :offset")
suspend fun getUsersPage(pageSize: Int, offset: Int): List<User>
```

### Q6: What are Room transactions?

**Answer:**

**Conceptual Understanding:**

A transaction in Room ensures that a set of database operations either all succeed or all fail. This maintains database consistency and prevents partial updates that could leave data in an inconsistent state.

**ACID Properties:**
- Atomicity: All operations succeed or all fail
- Consistency: Database rules are maintained
- Isolation: Concurrent transactions don't interfere
- Durability: Committed changes are permanent

**When to Use Transactions:**
- Multiple related database operations
- Transfer operations (debit/credit)
- Data integrity requirements
- Batch operations

```kotlin
@Dao
interface UserDao {
    
    @Transaction
    suspend fun transferPoints(fromUser: String, toUser: String, points: Int) {
        // Atomic operation - all or nothing
        decreasePoints(fromUser, points)
        increasePoints(toUser, points)
    }
    
    suspend fun decreasePoints(userId: String, points: Int) {
        // Implementation
    }
    
    suspend fun increasePoints(userId: String, points: Int) {
        // Implementation
    }
}
```

### Q7: How to encrypt Room database?

**Answer:**

**Conceptual Understanding:**

Encrypting a Room database protects sensitive data from unauthorized access, even if someone gains physical access to the device or extracted the database file.

**Why Encrypt Databases:**
- Protect sensitive user information
- Compliance with regulations (GDPR, HIPAA)
- Defense against device theft
- Prevent data extraction from rooted devices

**Encryption Approaches:**
- SQLCipher: Transparent encryption layer for SQLite
- File-level encryption: Encrypt the entire database file
- Field-level encryption: Encrypt specific sensitive fields

**Performance Considerations:**
- Encryption adds overhead (but usually minimal for mobile apps)
- Trade-off between security and performance
- Only encrypt when necessary

```kotlin
import androidx.sqlite.db.SupportSQLiteDatabase
import androidx.sqlite.db.framework.FrameworkSQLiteOpenHelperFactory
import net.sqlcipher.database.SupportFactory
import net.sqlcipher.database.SQLiteDatabase

class EncryptedDatabase {
    fun getEncryptedDatabase(context: Context): AppDatabase {
        val passphrase = SQLiteDatabase.getBytes("password".toCharArray())
        val factory = SupportFactory(passphrase)
        
        return Room.databaseBuilder(
            context.applicationContext,
            AppDatabase::class.java,
            "encrypted_database"
        )
            .openHelperFactory(factory)
            .build()
    }
}
```

### Q8: Explain Room queries and operations

**Answer:**

Use Room annotations to define type-safe SQL operations that are compile-time verified.
Prefer suspend and Flow-returning DAOs to integrate seamlessly with coroutines and reactive UIs.

**Conceptual Overview:**

Room provides several types of database operations through annotations, making database interactions type-safe and compile-time verified.

**Operation Categories:**
1. **Query**: Read operations (SELECT)
2. **Insert**: Add new data
3. **Update**: Modify existing data
4. **Delete**: Remove data
5. **Transaction**: Multiple operations atomically

**Query Features:**
- Compile-time validation (queries checked at compile time)
- Type safety (SQL errors caught during compilation)
- Kotlin Coroutines support (suspend functions)
- Reactive queries (LiveData/Flow)

**Query Types:**
```kotlin
@Dao
interface UserDao {
    
    // Simple query
    @Query("SELECT * FROM user")
    fun getAll(): Flow<List<User>>
    
    // Parameterized query
    @Query("SELECT * FROM user WHERE name = :name")
    suspend fun findByName(name: String): User?
    
    // Insert
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: User)
    
    // Update
    @Update
    suspend fun update(user: User)
    
    // Delete
    @Delete
    suspend fun delete(user: User)
    
    // Custom query
    @Query("UPDATE user SET name = :name WHERE uid = :uid")
    suspend fun updateName(uid: Int, name: String)
}
```

---

**Previous:** [Coroutines & Flows](coroutines-flows.md)

**Next:** [Networking](networking.md)

