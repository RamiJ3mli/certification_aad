# ROOM

[**Setup**](#Setup)

[**Primary components**](#Primary-components)

&nbsp;&nbsp;&nbsp;&nbsp;[Database configuration class](#Database-configuration-class)

&nbsp;&nbsp;&nbsp;&nbsp;[Entity class](#Entity-class)

&nbsp;&nbsp;&nbsp;&nbsp;[DAO class](#DAO-class)

&nbsp;&nbsp;&nbsp;&nbsp;[Database class](#Database-class)

<hr/>

## Setup

```guava
def room_version = "2.3.0"

implementation "androidx.room:room-runtime:$room_version"
kapt "androidx.room:room-compiler:$room_version"

// optional - Kotlin Extensions and Coroutines support for Room
implementation "androidx.room:room-ktx:$room_version"

// optional - Test helpers
testImplementation "androidx.room:room-testing:$room_version"
```

## Primary components

There are three major components in Room:

- The **database class** that holds the database and serves as the main access point for the underlying connection to your app's persisted data.

- **Data entities** that represent tables in your app's database.

- **Data access objects** (DAOs) that provide methods that your app can use to query, update, insert, and delete data in the database.

### Database configuration class

```kotlin
class AppDbConf {

    companion object {
        // db related constants
        internal const val DB_NAME = "AppDatabase.db"
        internal const val PDB_VERSION = 1

        // Contact table's columns
        internal const val CONTACT_TABLE_NAME = "contact"
        internal const val CONTACT_COL_ID = "id"
        internal const val CONTACT_COL_EMAIL = "email"
        internal const val CONTACT_COL_MOBILE_NUMBER = "mobile_number"
        internal const val CONTACT_COL_TEL_NUMBER = "telephone_number"

        // Version table's columns
        internal const val VERSION_TABLE_NAME = "data_version"
        internal const val VERSION_COL_ID = "id"
        internal const val VERSION_COL_VALUE = "version"
    }
}
```

### Entity class

```kotlin
@Entity
data class User(
    @PrimaryKey val uid: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

### DAO class

```kotlin
/**
 * This class serves as the base class for Room's dao components.
 * It provides the basic CRUD methods to keep the logic in child classes
 * to a minimum and avoid duplicated code.
 *
 * @param T The entity class on which we want to operate.
 */
abstract class AppDao<T> {

    /**
     * Inserts a single entity, or updates it if it has an existing primary key.
     *
     * @param entity The entity to upsert.
     * @return 1 if successful. 0 otherwise.
     */
    @Transaction
    open suspend fun insertOrUpdate(entity: T) =
        if (insert(entity) != -1L) 1 else update(entity)

    /**
     * Inserts a list of entities in bulk, or updates in bulk
     * if the entities have existing primary keys.
     *
     * @param entities The list of entities to upsert.
     * @return The number of rows affected if successful. 0 otherwise.
     */
    @Transaction
    open suspend fun insertOrUpdate(entities: List<T>): Int {
        if (entities.isEmpty()) return 0

        val rowIds = insert(entities)
        val insertCount = rowIds.filter { it != -1L }.size
        var updateCount = 0
        // Check if some entities were not inserted as they need updating instead
        if (rowIds.contains(-1L)) {
            // Get the indexes of unsuccessful inserts
            val updateIndexes = rowIds.mapIndexed { index, rowId ->
                if (rowId == -1L) index else null
            }.filterNotNull()

            // Get the entities to update
            val updateEntities = updateIndexes.map { entities[it] }

            // Update entities in bulk
            updateCount = update(updateEntities)
        }

        return insertCount + updateCount
    }

    /**
     * Inserts a single entity.
     *
     * @param entity The entity to insert.
     * @return The row id if successful.
     */
    @Insert(onConflict = OnConflictStrategy.IGNORE)
    abstract suspend fun insert(entity: T): Long

    /**
     * Inserts a list of entities in bulk.
     *
     * @param entities The list of entities to insert.
     * @return A list that contains a row id for every inserted entity.
     */
    @Insert(onConflict = OnConflictStrategy.IGNORE)
    abstract suspend fun insert(entities: List<T>): List<Long>

    /**
     * Updates a single entity.
     *
     * @param entity The entity to update.
     * @return The number of rows affected.
     */
    @Update
    abstract suspend fun update(entity: T): Int

    /**
     * Updates a list of entities in bulk.
     *
     * @param entities The list of entities to update.
     * @return The number of rows affected.
     */
    @Update
    abstract suspend fun update(entities: List<T>): Int

    /**
     * Deletes a single entity.
     *
     * @param entity The entity to delete.
     * @return The number of rows affected.
     */
    @Delete
    abstract suspend fun delete(entity: T): Int

    /**
     * Deletes a list of entities in bulk.
     *
     * @param entities The list of entities to Delete.
     * @return The number of rows affected.
     */
    @Delete
    abstract suspend fun delete(entities: List<T>): Int
}
```

```kotlin
@Dao
abstract class UserDao : AppDao<UserEntity>() {

    /**
     * Get contacts ordered by favorite and surname.
     */
    @Transaction
    @Query(
        "SELECT * FROM ${AppDbConf.CONTACT_TABLE_NAME}"
    )
    abstract fun getAll(): Flow<List<UserEntity>>

    /**
     * Get a single contact with the provided email.
     */
    @Transaction
    @Query(
        "SELECT * FROM ${AppDbConf.CONTACT_TABLE_NAME} " +
                "WHERE ${AppDbConf.CONTACT_COL_EMAIL} = :email " +
                "LIMIT 1"
    )
    abstract fun getByEmail(email: String): Flow<UserEntity?>
}
```

### Database class

```kotlin
@Database(
    entities = ContactEntity::class, VersionEntity::class],
    version = AppDatabase.DB_VERSION,
    exportSchema = true
)
abstract class AppDatabase : RoomDatabase() {

    abstract fun contactDao(): ContactDao
    abstract fun versionDao(): VersionDao

    companion object {

        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getAppDatabase(context: Context): AppDatabase = synchronized(this) {
            INSTANCE ?: buildDatabase(context).also { INSTANCE = it }
        }

        private fun buildDatabase(context: Context) = Room.databaseBuilder(                         
            context.applicationContext,
            AppDatabase::class.java,
            AppDatabaseConf.DB_NAME
        ).build()
    }
}
```
