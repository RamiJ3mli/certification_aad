# NOTIFICATIONS
## Notification APIs
Almost with each release of Android, the notification APIs change. So, we have to use [`NotificationCompat`](https://developer.android.com/reference/androidx/core/app/NotificationCompat) and its subclasses, as well as [`NotificationManagerCompat`](https://developer.android.com/reference/androidx/core/app/NotificationManagerCompat) in our app. This will help us avoid writing conditional code to check API levels. `NotificationCompat` is updated as the platform evolves to include the latest methods.

```
const val CHANNEL_1_ID = "channel1"
const val CHANNEL_2_ID = "channel2"

class App : Application() {

    private var notificationManager: NotificationManagerCompat? = null

    override fun onCreate() {
        super.onCreate()
        createNotificationChannels()
    }

    private fun createNotificationChannels() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            // Create notification channel 1
            val channel1 = NotificationChannel(
                CHANNEL_1_ID,
                "Channel 1",
                NotificationManager.IMPORTANCE_HIGH
            )
            channel1.description = "This is Channel 1"

            // Create notification channel 2
            val channel2 = NotificationChannel(
                CHANNEL_2_ID,
                "Channel 2",
                NotificationManager.IMPORTANCE_LOW
            )
            channel2.description = "This is Channel 2"

            // Enable the notification channels
            notificationManager = NotificationManagerCompat.from(this).apply {
                createNotificationChannel(channel1)
                createNotificationChannel(channel2)
            }
        }
    }

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
        notificationManager?.notify(1, notification)
    }
}
```
