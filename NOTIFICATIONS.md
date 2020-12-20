# NOTIFICATIONS
Almost with each release of Android, the notification APIs change. So, we have to use [`NotificationCompat`](https://developer.android.com/reference/androidx/core/app/NotificationCompat) and its subclasses, as well as [`NotificationManagerCompat`](https://developer.android.com/reference/androidx/core/app/NotificationManagerCompat) in our app. This will help us avoid writing conditional code to check API levels. `NotificationCompat` is updated as the platform evolves to include the latest methods.

## Create a notification channel

```kotlin
const val CHANNEL_ID = "CHANNEL_ID"

private fun createNotificationChannels() {
    // Notifications channels are supported starting from android 8, API 26
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.O) {
        return
    }
    
    // Create a notification channel
    val channel = NotificationChannel(CHANNEL_ID,"Channel name",NotificationManager.IMPORTANCE_HIGH)
    channel.description = "This is an epic Channel!"

    // Enable the notification channel
    NotificationManagerCompat.from(this).apply {
        createNotificationChannel(channel)
    }
}
```
## Show a notification

```kotlin
// Show a notification
fun sendNotification() {
    val title = "Notification title"
    val message = "Notification message"
    val notification: Notification = NotificationCompat.Builder(this, CHANNEL_ID)
        .setSmallIcon(R.drawable.btn_star)
        .setContentTitle(title)
        .setContentText(message)
        
        // Set the notification's color
        .setColor(Color.BLUE)
        
        // Set the color specified in setColor as the notification's background color
        .setColorize(true)
        
        // Make this notification automatically dismissed when tapped
        .setAutoCancel(true)
        
        // Set this flag if we would only like the sound, vibration and ticker to be played if the notification is not already showing
        .setOnlyAlertOnce(true)
        
        // Set the intrusion level of the notification for Android 7.1 and lower. Use channel importance for Android 8 and higher.
        .setPriority(NotificationCompat.PRIORITY_HIGH)
        .setCategory(NotificationCompat.CATEGORY_MESSAGE)
        .build()
    notificationManager?.notify(NOTIFICATION_ID, notification)
}
```
## Show a notification that opens an activity when tapped
```kotlin
// Show a notification that opens an activity when tapped
fun sendNotification() {
    // Create an activity intent
    val activityIntent = Intent(this, MainActivity::class.java).apply {
        flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
    }
    // Wrap it in a pending intent
    val contentIntent = PendingIntent.getActivity(this, 0, activityIntent, 0)
    
    // Create a notification and submit it
    val notification: Notification = NotificationCompat.Builder(this, CHANNEL_ID)
        ..
        .setContentIntent(contentIntent)
        ..
        .build()
    notificationManager?.notify(NOTIFICATION_ID, notification)
}
```
## Show a notification with actions
```kotlin
// Show a notification with an action button that invokes a broadcast receiver
fun sendNotification() {
    // Create an intent for our broadcast receiver
    val broadcastIntent = Intent(this, NotificationReceiver::class.java)
    // Pass data to the broadcast receiver
    broadcastIntent.putExtra(ARG_NAME, message)
   
    // Wrap it in a pending intent
    // FLAG_UPDATE_CURRENT is used to the data in the extra argument if needed
    val actionIntent = PendingIntent.getBroadcast(this,0, broadcastIntent, PendingIntent.FLAG_UPDATE_CURRENT)
    
    // Create a notification and submit it
    val notification: Notification = NotificationCompat.Builder(this, CHANNEL_ID)
        ..
        .addAction(R.mipmap.ic_launcher, "Toast", actionIntent))
        ..
        .build()
    notificationManager?.notify(NOTIFICATION_ID, notification)
}
```
**Notification receiver**
```kotlin
// 
class NotificationReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val message = intent.getStringExtra(ARG_NAME)
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show()
    }
}
```
**AndroidManifest.xml**
```xml
<receiver android:name=".NotificationReceiver" />
```
## Notification styles
### Show a notification with BigPictureStyle
```kotlin
// Show a notification with BigPictureStyle
fun sendNotification() {
    // Get bitmap from drawable 
    val largeIcon = BitmapFactory.decodeResource(resources, R.drawable.icon_x)    
   
   // Set up style
   val notificationStyle = NotificationCompat.BigPictureStyle()
                .bigPicture(largeIcon)
                // To make the image appear as a thumbnail only when the notification is collapsed
                .bigLargeIcon(null)
                
    // Create a notification and submit it
    val notification: Notification = NotificationCompat.Builder(this, CHANNEL_ID)
        ..
        // To make the image appear as a thumbnail only when the notification is collapsed
        .setLargeIcon(largeIcon)
        .setStyle(notificationStyle)
        ..
        .build()
    notificationManager?.notify(NOTIFICATION_ID, notification)
}
```



???????
addAction(Action action)
