# PAGING

[**Setup**](#Setup)

[**Implementation**](#Implementation)

&nbsp;&nbsp;&nbsp;&nbsp;[Datastore configuration](#Datastore-configuration)

&nbsp;&nbsp;&nbsp;&nbsp;[Datastore class](#Datastore-class)

&nbsp;&nbsp;&nbsp;&nbsp;[Store/read key-value pairs](#Store/read-key-value-pairs)

<hr/>

## Setup

```guava
implementation 'androidx.datastore:datastore-preferences:1.0.0-beta01'
```

## Implementation

### Datastore configuration

```kotlin
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.doublePreferencesKey
import androidx.datastore.preferences.core.floatPreferencesKey
import androidx.datastore.preferences.core.intPreferencesKey
import androidx.datastore.preferences.core.longPreferencesKey
import androidx.datastore.preferences.core.stringPreferencesKey

class AppDatastoreConf {

    companion object {
        internal const val PREF_NAME = "settings"
        internal val COUNTER = intPreferencesKey("counter")
        internal val BIG_COUNTER = longPreferencesKey("counter")
        internal val WEIGHT = floatPreferencesKey("weight")
        internal val LENGTH = doublePreferencesKey("length")
        internal val USERNAME = stringPreferencesKey("username")
        internal val isEmployee = booleanPreferencesKey("is_employee")
    }
}
```

### Datastore class

```kotlin
import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.PreferenceDataStoreFactory
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.preferencesDataStoreFile

class AppDatastore {

    companion object {

        @Volatile
        private var INSTANCE: DataStore<Preferences>? = null

        fun getDataStore(context: Context, name: String): DataStore<Preferences> =
            synchronized(this) {
                INSTANCE ?: buildDataStore(context, name).also { INSTANCE = it }
            }

        private fun buildDataStore(context: Context, name: String) =
            PreferenceDataStoreFactory.create {
                context.preferencesDataStoreFile(name)
            }
    }
}
```

### Store/read key-value pairs

```kotlin
class MainActivity : AppCompatActivity(R.layout.activity_main) {

    private val dataStore = AppDatastore.getDataStore(
        applicationContext,
        AppDatastoreConf.PREF_NAME
    )

    fun store(view: View) {
        lifecycleScope.launch(Dispatchers.IO) {
            dataStore.edit { datastore ->
                val currentCounterValue = datastore[AppDatastoreConf.COUNTER] ?: 0
                datastore[AppDatastoreConf.COUNTER] = currentCounterValue + 1
            }
        }
    }

    fun read(view: View) {
        lifecycleScope.launch(Dispatchers.IO) {
            dataStore.data
                .map { preferences -> preferences[AppDatastoreConf.COUNTER] ?: 0 }
                .collectLatest {
                    withContext(Dispatchers.Main) {
                        Toast.makeText(applicationContext, "$it", Toast.LENGTH_SHORT).show()
                    }
                }
        }
    }
}
```
