# NOTIFICATIONS
## Notification APIs
Almost with each release of Android, the notification APIs change. So, we have to use [`NotificationCompat`](https://developer.android.com/reference/androidx/core/app/NotificationCompat) and its subclasses, as well as [`NotificationManagerCompat`](https://developer.android.com/reference/androidx/core/app/NotificationManagerCompat) in our app. This will help us avoid writing conditional code to check API levels. `NotificationCompat` is updated as the platform evolves to include the latest methods.

### Android 5.0, API level 21
-   Introduced lock screen and heads-up notifications.
-   The user can now put the phone into Do Not Disturb mode and configure which notifications are allowed to interrupt them when the device is in priority only mode.
-   Methods added to API set whether or not a notification is displayed on the lock screen ([`setVisibility()`](https://developer.android.com/reference/androidx/core/app/NotificationCompat.Builder#setVisibility(int))) and for specifying “public” version of the notification text.
-   [`setPriority()`](https://developer.android.com/reference/androidx/core/app/NotificationCompat.Builder#setPriority(int)) method added which tells the system how “interruptive” this notification should be (e.g. setting it to high makes the notification appear as a heads-up notification).
-   Notification stacks support added to Android Wear (now called Wear OS) devices. Put notifications into a stack using [`setGroup()`](https://developer.android.com/reference/androidx/core/app/NotificationCompat.Builder#setGroup(java.lang.String)). Note that notification stacks were not supported on tablets nor phones yet. Notification stacks would later become known as a group or bundle.
### Android 7.0, API level 24
-   Notification templates were restyled to put emphasis on the hero image and avatar.
-   Three notification templates were added: one for messaging apps and the other two for decorating custom content views with the expandable affordance and other system decorations.
-   Support added to handheld devices (phones and tablets) for notification groups. Uses the same API as Android Wear (now called Wear OS) notification stacks introduced in Android 5.0 (API level 21).
-   Users can reply directly inside of a notification (they can enter text which will then be routed to the notification’s parent app) using inline reply.

### Android 8.0, API level 26
-   Individual notifications must now be put in a specific channel.
-   Users can now turn off notifications per  [channel](https://developer.android.com/guide/topics/ui/notifiers/notifications#ManageChannels), instead of turning off all notifications from an app.
-   Apps with active notifications display a notification "badge" on top of their app icon on the home/launcher screen.
-   Users can now snooze a notification from the drawer. You can set an automatic timeout for a notification.
-   You can also set the notification's background color.
-   Some APIs regarding notification behaviors were moved from [`Notification`](https://developer.android.com/reference/android/app/Notification) to  [`NotificationChannel`](https://developer.android.com/reference/android/app/NotificationChannel). For example, use [`NotificationChannel.setImportance()`](https://developer.android.com/reference/android/app/NotificationChannel#setImportance(int)) instead of [`NotificationCompat.Builder.setPriority()`](https://developer.android.com/reference/androidx/core/app/NotificationCompat.Builder#setPriority(int)) for Android 8.0 and higher.
