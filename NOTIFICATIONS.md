# NOTIFICATIONS
Almost with each release of Android, the notification APIs change. So, we have to use [`NotificationCompat`](https://developer.android.com/reference/androidx/core/app/NotificationCompat) and its subclasses, as well as [`NotificationManagerCompat`](https://developer.android.com/reference/androidx/core/app/NotificationManagerCompat) in our app. This will help us avoid writing conditional code to check API levels. `NotificationCompat` is updated as the platform evolves to include the latest methods.

## Create a notification channel

```
const val CHANNEL_ID = "CHANNEL_ID"

private fun createNotificationChannels() {
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.O) {
        return
    }
    
    // Create a notification channel
    val channel = NotificationChannel(
        CHANNEL_ID,
        "Channel name",
        NotificationManager.IMPORTANCE_HIGH
    )
    channel.description = "This is an epic Channel!"

    // Enable the notification channel
    NotificationManagerCompat.from(this).apply {
        createNotificationChannel(channel)
    }
}
```
## Show a notification

```
// Show notification
fun sendNotification(channelId: String) {
    val title = "Notification title"
    val message = "Notification message"
    val notification: Notification = NotificationCompat.Builder(this, channelId)
        .setSmallIcon(R.drawable.btn_star)
        .setContentTitle(title)
        .setContentText(message)
        .setPriority(NotificationCompat.PRIORITY_HIGH)
        .setCategory(NotificationCompat.CATEGORY_MESSAGE)
        .build()
    notificationManager?.notify(NOTIFICATION_ID, notification)
}
```
## Open activity when notification is tapped
```
// Show notification
fun sendNotification(channelId: String) {
    // Create an activity intent
    val activityIntent = Intent(this, MainActivity::class.java)
    // Wrap it in a pending intent
    val contentIntent = PendingIntent.getActivity(this, 0, activityIntent, 0)
    
    // Create a notification and submit it
    val notification: Notification = NotificationCompat.Builder(this, channelId)
        ..
        .setContentIntent(contentIntent)
        ..
        .build()
    notificationManager?.notify(NOTIFICATION_ID, notification)
}
```


