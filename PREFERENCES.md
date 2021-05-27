# PAGING

[**Setup**](#Setup)

[**Implementation**](#Implementation)

&nbsp;&nbsp;&nbsp;&nbsp;[Settings activity](#Settings-activity)

&nbsp;&nbsp;&nbsp;&nbsp;[Settings fragment](#Settings-activity)

<hr/>

## Setup

```guava
implementation 'androidx.preference:preference-ktx:1.1.1'
```

## Implementation

### Settings activity

```xml
<androidx.fragment.app.FragmentContainerView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/settingsNavHost"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:defaultNavHost="true"
    app:navGraph="@navigation/settings_graph"
    android:name="androidx.navigation.fragment.NavHostFragment" />
```

```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/settings_graph"
    app:startDestination="@id/settingsFragment">

    <fragment
        android:id="@+id/settingsFragment"
        android:name="com.ramijemli.preferences.SettingsFragment"
        android:label="SettingsFragment" >
        <action
            android:id="@+id/goToAccountsFragment"
            app:destination="@id/accountsFragment" />
    </fragment>

    <fragment
        android:id="@+id/accountsFragment"
        android:name="com.ramijemli.preferences.AccountsFragment"
        android:label="AccountsFragment" />

</navigation>
```

```kotlin
class SettingsActivity : AppCompatActivity(R.layout.settings_activity),
    PreferenceFragmentCompat.OnPreferenceStartFragmentCallback {

    // For preference items that navigate to a new fragment
    override fun onPreferenceStartFragment(
        caller: PreferenceFragmentCompat?,
        preference: Preference?
    ): Boolean {
        preference ?: return false

        if (preference.fragment == AccountsFragment::class.qualifiedName) {
            findNavController(R.id.settingsNavHost).navigate(R.id.goToAccountsFragment)
        }

        return true
    }
}
```

### Settings fragment

```xml
// arrays.xml
<string-array name="reply_entries">
    <item>Reply</item>
    <item>Reply to all</item>
</string-array>

<string-array name="reply_values">
    <item>reply</item>
    <item>reply_all</item>
</string-array>
```

```xml
// settings_preferences.xml in xml folder
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <PreferenceCategory app:title="App Widget">

        <CheckBoxPreference
            app:icon="@drawable/ic_launcher_background"
            app:key="resizable"
            app:summary="Make the app widget resizable"
            app:title="Resizable" />

    </PreferenceCategory>

    <PreferenceCategory app:title="Display">

        <DropDownPreference
            app:entries="@array/dark_mode_behavior_titles"
            app:entryValues="@array/dark_mode_behavior_values"
            app:icon="@drawable/ic_launcher_background"
            app:key="dark_mode_behavior"
            app:title="Dark mode behavior"
            app:useSimpleSummaryProvider="true" />

        <SeekBarPreference
            android:max="10"
            app:defaultValue="5"
            app:iconSpaceReserved="false"
            app:key="display_brightness"
            app:showSeekBarValue="true"
            app:title="Display brightness" />

        <SwitchPreferenceCompat
            app:icon="@drawable/ic_launcher_background"
            app:key="dark_mode"
            app:summaryOff="Dark mode is disabled"
            app:summaryOn="Dark mode is enabled"
            app:title="Dark mode" />

    </PreferenceCategory>

    <PreferenceCategory app:title="Notifications">

        <EditTextPreference
            app:dialogIcon="@mipmap/ic_launcher"
            app:dialogMessage="Yes. This is a message."
            app:dialogTitle="Set up user signature"
            app:icon="@drawable/ic_launcher_background"
            app:key="signature"
            app:title="Signature"
            app:useSimpleSummaryProvider="true" />

        <ListPreference
            app:defaultValue="reply"
            app:entries="@array/reply_entries"
            app:entryValues="@array/reply_values"
            app:icon="@drawable/ic_launcher_background"
            app:key="reply"
            app:negativeButtonText="Nah"
            app:positiveButtonText="Sure"
            app:title="@string/reply_title"
            app:useSimpleSummaryProvider="true" />

        <MultiSelectListPreference
            app:entries="@array/reply_entries"
            app:entryValues="@array/reply_values"
            app:icon="@drawable/ic_launcher_background"
            app:key="multi_select_list"
            app:title="Some multi select list"
            app:useSimpleSummaryProvider="true" />

    </PreferenceCategory>

    <PreferenceCategory
        app:initialExpandedChildrenCount="1"
        app:title="Sync">

        <SwitchPreferenceCompat
            app:disableDependentsState="false"
            app:icon="@drawable/ic_launcher_background"
            app:key="dark_mode2"
            app:shouldDisableView="true"
            app:summaryOff="Dark mode is disabled"
            app:summaryOn="Dark mode is enabled"
            app:title="Dark mode" />

        <SwitchPreferenceCompat
            app:dependency="dark_mode2"
            app:icon="@drawable/ic_launcher_background"
            app:key="dark_mode3"
            app:summaryOff="Dark mode is disabled"
            app:summaryOn="Dark mode is enabled"
            app:title="Dark mode" />

        <SwitchPreferenceCompat
            app:dependency="dark_mode2"
            app:icon="@drawable/ic_launcher_background"
            app:key="dark_mode4"
            app:summaryOff="Dark mode is disabled"
            app:summaryOn="Dark mode is enabled"
            app:title="Dark mode" />

        <SwitchPreferenceCompat
            app:dependency="dark_mode2"
            app:icon="@drawable/ic_launcher_background"
            app:key="dark_mode5"
            app:summaryOff="Dark mode is disabled"
            app:summaryOn="Dark mode is enabled"
            app:title="Dark mode" />

    </PreferenceCategory>

    <PreferenceCategory app:title="Accounts">

        <Preference
            app:key="preference"
            app:icon="@drawable/ic_launcher_background"
            app:summary="Tap to toast!"
            app:title="Custom click listener" />

        <Preference
            app:fragment="com.ramijemli.preferences.AccountsFragment"
            app:icon="@drawable/ic_launcher_background"
            app:summary="Set up the device's default account"
            app:title="Add account" />

        <Preference
            app:icon="@drawable/ic_launcher_background"
            app:key="activity"
            app:summary="Launch activity"
            app:title="Launch activity">

            <intent
                android:targetClass="com.ramijemli.preferences.SettingsActivity"
                android:targetPackage="com.ramijemli.preferences">

                <extra
                    android:name="example_key"
                    android:value="example_value" />
            </intent>
        </Preference>

        <Preference
            app:icon="@drawable/ic_launcher_background"
            app:key="activity_link"
            app:summary="Launch URL"
            app:title="Launch URL">

            <intent
                android:action="android.intent.action.VIEW"
                android:data="http://www.google.com" />
        </Preference>

    </PreferenceCategory>

</PreferenceScreen>
```

```kotlin
import android.os.Bundle
import android.text.InputType
import android.widget.Toast
import androidx.preference.EditTextPreference
import androidx.preference.Preference
import androidx.preference.PreferenceDataStore
import androidx.preference.PreferenceFragmentCompat

class SettingsFragment : PreferenceFragmentCompat() {
    override fun onCreatePreferences(savedInstanceState: Bundle?, rootKey: String?) {
        setPreferencesFromResource(R.xml.settings_preferences, rootKey)

        // Create preferences programmatically
        // val context = preferenceManager.context
        // val screen = preferenceManager.createPreferenceScreen(context)

        // val notificationPreference = SwitchPreferenceCompat(context).apply {
        //     key = "notifications"
        //     title = "Enable message notifications"
        // }

        // val notificationCategory = PreferenceCategory(context).apply {
        //     key = "notifications_category"
        //     title = "Notifications"
        //     addPreference(notificationPreference)
        // }
        // screen.addPreference(notificationCategory)

        // preferenceScreen = screen

        preferenceManager.preferenceDataStore = object: PreferenceDataStore(){
            override fun putString(key: String, value: String?) {
                // Save the value somewhere
            }

            override fun getString(key: String, defValue: String?): String? {
                // Retrieve the value
                return ""
            }
        }

        val preferenceItem: Preference? = findPreference("preference")
        preferenceItem?.setOnPreferenceClickListener {
            Toast.makeText(preferenceManager.context, "Tapped", Toast.LENGTH_SHORT).show()
            true
        }

        val editTextPreference: EditTextPreference? = findPreference("signature")

        editTextPreference?.setOnBindEditTextListener { editText ->
            editText.inputType = InputType.TYPE_CLASS_NUMBER
        }

        editTextPreference?.summaryProvider =
            Preference.SummaryProvider<EditTextPreference> { preference ->
                val username = preference.text
                if (username.isEmpty()) {
                    "Custom not set!"
                } else {
                    "Length of saved value: " + username.length
                }
            }
    }
}
```
