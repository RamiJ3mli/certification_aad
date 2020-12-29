# BROADCAST RECEIVERS
 Android apps can send or receive broadcast messages from the Android system and other Android apps, similar to the publish-subscribe design pattern. These broadcasts are sent when an event of interest occurs like for example when the system boots up or the device starts charging. We can use **Broadcast Receivers** to listen to these events.

## Receiving broadcasts
Apps can receive broadcasts in two ways: through manifest-declared receivers and context-registered receivers.

### Manifest-declared receivers
If you declare a broadcast receiver in your manifest, the system launches your app (if the app is not already running) when the broadcast is sent.

```xml
<receiver android:name=".MyBroadcastReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <action android:name="android.intent.action.INPUT_METHOD_CHANGED" />
    </intent-filter>
</receiver>
```
**Specifying an intent filter in manifest will set the BR's exported state to true. otherwise, it's set to false.**

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context, "Broadcast received!", Toast.LENGTH_SHORT).show()
    }
}
```

### Context-registered receivers
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
In some cases, we may need to trigger the BRs of our app in a specific order. For example, after an event of interest is triggered, we may want to update an activity if the app is visibile, or show a notification otherwise.

By setting the android:priority attribute of the matching intent-filter, we can control the execution order of the BRs. BRs with the same priority will be run in an arbitrary order.
```xml
<receiver android:name=".MyBroadcastReceiver1">
    <intent-filter android:priority="3">
        <action android:name="com.ramijemli.ACTION_NAME" />
    </intent-filter>
</receiver>

<receiver android:name=".MyBroadcastReceiver2">
    <intent-filter android:priority="2">
        <action android:name="com.ramijemli.ACTION_NAME"/>
    </intent-filter>
</receiver>
```
```kotlin
val filter = IntentFilter("com.ramijemli.ACTION_NAME").apply {
    setPriority(1)
}
registerReceiver(broadcastReceiver3, filter)

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