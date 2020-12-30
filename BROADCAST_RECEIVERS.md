## [Emulate a system broadcast using command line](#Emulate-a-system-broadcast-using-command-line)

## [Receiving broadcasts](#Receiving-broadcasts)
### [Manifest-declared receivers (static)](#Manifest-declared-receivers-(static))
#### [Disable manifest-declared receivers](#Disable-manifest-declared-receivers)
### [Context-registered receivers (dynamic)](#Context-registered-receivers-(dynamic))
### [Sticky broadcasts](#Sticky-broadcasts)

## [Sending broadcasts](#Sending-broadcasts)
### [Sending implicit broadcasts](#Sending-implicit-broadcasts)
### [Sending explicit broadcasts](#Sending-explicit-broadcasts)
### [Sending ordered broadcasts](#Sending-ordered-broadcasts)
### [Passing data between ordered broadcast receivers](#Passing-data-between-ordered-broadcast-receivers)

## [Local broadcasts](#Local-broadcasts)

## [Going async](#Going-async)

## [Securing broadcasts with permissions](#Securing-broadcasts-with-permissions)
### [Sending broadcasts with permissions](#Sending-broadcasts-with-permissions)
### [Receiving broadcasts with permissions](#Receiving-broadcasts-with-permissions)

<hr/>

# BROADCAST RECEIVERS
 Android apps can send or receive broadcast messages from the Android system and other Android apps, similar to the publish-subscribe design pattern. These broadcasts are sent when an event of interest occurs like for example when the system boots up or the device starts charging. We can use **Broadcast Receivers** to listen to these events.

## Emulate a system broadcast using command line

```shell
# trigger a broadcast and deliver it to a component
adb shell am activity/service/broadcast -a ACTION -c CATEGORY -n NAME

# for example (this goes into one line)
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED -c android.intent.category.HOME -n package_name/class_name
```

## Receiving broadcasts
Apps can receive broadcasts in two ways: through manifest-declared receivers and context-registered receivers.

### Manifest-declared receivers (static)
If you declare a broadcast receiver in your manifest, the system launches your app (if the app is not already running) when the broadcast is sent.

```xml
<receiver android:name=".MyBroadcastReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <action android:name="android.intent.action.INPUT_METHOD_CHANGED" />
    </intent-filter>
</receiver>
```
> Specifying an intent filter in manifest will set the BR's exported state to true. otherwise, it's set to false.

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context, "Broadcast received!", Toast.LENGTH_SHORT).show()
    }
}
```

#### Disable manifest-declared receivers

```kotlin
val receiver = ComponentName(context, MyBroadcastReceiver.class);
val pm = context.getPackageManager();

pm.setComponentEnabledSetting(receiver, PackageManager.COMPONENT_ENABLED_STATE_DISABLED, PackageManager.DONT_KILL_APP);
```

### Context-registered receivers (dynamic)
This type of BR always receives data upon registration which is why it's called sticky.<br/>
To register a receiver with a context, perform the following steps:

1. Create an instance of BroadcastReceiver.
2. Create an IntentFilter and register the receiver by calling registerReceiver(BroadcastReceiver, IntentFilter)

```kotlin
val br: BroadcastReceiver = MyBroadcastReceiver()

override fun onStart() {
    super.onStart()
    val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
    registerReceiver(br, filter)
}
 
override fun onStop() {
    super.onResume()
    unregisterReceiver(br)
}
```

### Sticky broadcasts
The Android system uses sticky broadcast for certain system information. For example, the battery status is send as sticky intent and can get received at any time. The following example demonstrates that.

```kotlin
// Register for the battery changed event
val filter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)

// Intent is sticky so using null as receiver works fine
// Returned value contains the status
val batteryStatus = context.registerReceiver(null, filter)

// Are we charging / charged?
val status = batteryStatus.getIntExtra(BatteryManager.EXTRA_STATUS, -1)
val isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING || status == BatteryManager.BATTERY_STATUS_FULL

val isFull = status == BatteryManager.BATTERY_STATUS_FULL;

// How are we charging?
val chargePlug = batteryStatus.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1)
val usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB
val acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC
```

## Sending broadcasts

### Sending implicit broadcasts
A broadcast that is sent by just providing the event type(intent action) is called an implicit broadcast. Any app that has an intent filter corresponding to the broadcasted intent action can be triggered.

```kotlin
fun sendBroadcast() {
    val intent = Intent("com.ramijemli.ACTION_NAME").apply {
        putExtra("com.ramijemli.EXTRA_ARG", "Broadcast received")
    }
    sendBroadcast(intent)
}
```

### Sending explicit broadcasts
A broadcast that can specifically target a BR class of another app is called an explicit broadcast. Except the target app, no other one can detect the broadcast because we use the package name, or even the BR's classname directly.

```kotlin
fun sendBroadcast() {
    // Send explicit broadcast within the same app
    Intent().apply {
        setClass(this, MyBroadcastReceiver::java.class)
        sendBroadcast(this)
    }

    // Send explicit broadcast to call directly an external BR
    Intent().apply {
        val cn = ComponentName("com.ramijemli.appid", "com.ramijemli.appid.MyBroadcastReceiver")
        setComponent(cn)
        sendBroadcast(this)
    }

    // Send explicit broadcast to call directly an external BR
    Intent().apply {
        setClassName("com.ramijemli.appid", "com.ramijemli.appid.MyBroadcastReceiver")
        sendBroadcast(this)
    }

    // Send explicit broadcast with an intent action but for BRs of a specific app.
    // The BRs of the target app have to specify a corresponding intent filter
    Intent("com.ramijemli.ACTION_NAME").apply {
        setPackage("com.ramijemli.appid")
        sendBroadcast(this)
    }

    // Send explicit broadcast by looking up the external BRs available
    Intent("com.ramijemli.ACTION_NAME").apply {
        val packageManager = getPackageManager()
        val infos = packageManager.queryBroadcastReceivers(it, 0)

        infos.forEach { info -> {
            // activityInfo corresponds the broadcast receiver class
            val cn = ComponentName(info.activityInfo.packageName, info.activityInfo.name)
            it.setComponent(cn)
            sendBroadcast(it)
        }}
    }
}
```

### Sending ordered broadcasts
In some cases, we may need to trigger the BRs of our app in a specific order. For example, after an event of interest is triggered, we want to invoke a dynamic BR to update an activity if the app is visibile, and then show a notification. If the dynamic BR is not registered, we want to execute only the second step.

By setting the android:priority attribute of the matching intent-filter, we can control the execution order of the BRs. BRs with the same priority will be run in an arbitrary order.

```xml
<receiver android:name=".NotificationReceiver">
    <intent-filter android:priority="1">
        <action android:name="com.ramijemli.ACTION_NAME" />
    </intent-filter>
</receiver>
```

```kotlin
val filter = IntentFilter("com.ramijemli.ACTION_NAME").apply {
    setPriority(2)
}
registerReceiver(UpdateReceiver, filter)
```

```kotlin
fun sendOrderedBroadcast() {
    // Send an ordered explicit broadcast with an intent action but for BRs of a specific app.
    // The BRs of the target app have to specify a corresponding intent filter
    Intent("com.ramijemli.ACTION_NAME").apply {
        setPackage("com.ramijemli.appid")
        sendOrderedBroadcast(this, null)
    }
}
```

The BRs will be triggered according to the priority attribute's descending order.

### Passing data between ordered broadcast receivers
Using the sendOrderedBroadcast method, we can pass data to a BR. (A result code, an initial data as String, and a bundle)

```kotlin
fun sendOrderedBroadcast() {
    val extras = Bundle().apply {
            putString(TITLE_ARG, "Username")
            putString(REPLY_ARG, "Reply")
    }

    // Send an ordered explicit broadcast with an intent action but for BRs of a specific app.
    // The BRs of the target app have to specify a corresponding intent filter
    Intent("com.ramijemli.ACTION_NAME").apply {
        setPackage("com.ramijemli.appid")
        sendOrderedBroadcast(this, receiverPermission = null, FinalReceiver(), scheduler = null, initialCode = 0, initialData = "initialData", extras)
    }
}
```

```kotlin
class UpdateReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        val resultCode: Int = getResultCode()
        val resultData: String = getResultData()
        
        val resultExtras: Bundle = getResultExtras(true)
        val title = resultExtras.getString(TITLE_ARG)
        val replyText = resultExtras.getString(REPLY_ARG)

        /** Update activity **/

        // As the activity was updated, we don't need to run the notification receiver.
        // So, we abort the broadcast
        abortBroadcast()

        // We still can pass data to the notification receiver
        setResult(Activity.RESULT_OK, resultData, resultExtras);
    }
}
```

```kotlin
class NotificationReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        val resultCode: Int = getResultCode()
        val resultData: String = getResultData()
        
        val resultExtras: Bundle = getResultExtras(true)
        val title = resultExtras.getString(TITLE_ARG)
        val replyText = resultExtras.getString(REPLY_ARG)

        /** Show notification **/

        // We still can pass data to the sender receiver
        setResult(Activity.RESULT_OK, resultData, resultExtras);
    }
}
```

> `setResult`, `setResultCode`, `setResultData`, and `setResultExtras` doesn't work with non-ordered broadcasts.

## Local broadcasts
If we want events of interest to be broadcasted and received internally only (inside our app) without worrying about exposing senstive data, we can use the `LocalBroadcastManager` class.

```kotlin
val br: BroadcastReceiver = MyBroadcastReceiver()
var localBroadcatManager: LocalBroadcastManager? = null

override fun onCreate(savedInstanceState: Bundle?) {
    ..
    localBroadcatManager = LocalBroadcastManager.getInstance(this)
}

override fun onStart() {
    super.onStart()
    val filter = IntentFilter("com.ramijemli.ACTION_NAME")
    localBroadcatManager?.registerReceiver(br, filter)
}
 
override fun onStop() {
    super.onResume()
    localBroadcatManager?.unregisterReceiver(br)
}

fun sendBroadcast(View v) {
    val intent = new Intent("com.ramijemli.ACTION_NAME")
    localBroadcastManager.sendBroadcast(intent)
}
```

`LocalBroadcastManager` can only work with context-registered receivers. The static ones(manifest) will never be triggered.

## Going async
In some cases, we may need to perform asynchronous work when `onReceive` is called. We can use `goAsync` to get a `PendingResult` object, and call the latter's `finish` method when the processing is done. Keep in mind that we have under 10 seconds at most before the system kills the entire process. It's recommended to use other suitable APIs like workmanager and services.

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        val pendingResult: PendingResult = goAsync()

        // Must call finish() inside the coroutine when work is done,
        // so the BroadcastReceiver can be recycled.
        pendingResult.finish()
    }
}
```

## Securing broadcasts with permissions
Permissions allow us to restrict broadcasts to the set of apps that hold certain permissions. We can enforce restrictions on either the sender or receiver of a broadcast.

### Sending broadcasts with permissions
We can specify a permission parameter when we use `sendBroadcast` or `sendOrderedBroadcast`. This way only receivers who have requested that permission with the `uses-permission` tag in their manifest (and subsequently been granted the permission if it is dangerous) can receive the broadcast. 

```kotlin
// Sender app
sendBroadcast(Intent("com.ramijemli.ACTION_NAME"), "com.ramijemli.appid.CUSTOM_PERMISSION")
```

```xml
<!-- Sender app -->
<permission android:name="com.ramijemli.appid.CUSTOM_PERMISSION"/>
```

To receive the broadcast, the receiving app must request the permission.

```xml
<!-- Receiver app -->
<uses-permission android:name="android.permission.SEND_SMS"/>
```

### Receiving broadcasts with permissions
We can specify a permission parameter when registering a broadcast receiver either with `registerReceiver` or in the `<receiver>` tag in our manifest, then only broadcasters who have requested the permission with the <uses-permission> tag in their manifest (and subsequently been granted the permission if it is dangerous) can send an Intent to the receiver.

```xml
<!-- Receiver app -->
<permission android:name="com.ramijemli.appid.CUSTOM_PERMISSION"/>

<receiver android:name=".MyBroadcastReceiver"
          android:permission="com.ramijemli.appid.CUSTOM_PERMISSION">
    <intent-filter>
        <action android:name="android.intent.action.AIRPLANE_MODE"/>
    </intent-filter>
</receiver>
```

Or our receiver app has a context-registered receiver.

```kotlin
// Receiver app
var filter = IntentFilter("com.ramijemli.ACTION_NAME")
registerReceiver(receiver, filter, "com.ramijemli.appid.CUSTOM_PERMISSION", null)
```

Then, to be able to send broadcasts to those receivers, the sending app must request the permission.

```xml
<!-- Sender app -->
<uses-permission android:name="com.ramijemli.appid.CUSTOM_PERMISSION"/>
```
