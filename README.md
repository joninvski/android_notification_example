Android notification example
============================

<img src="https://github.com/joninvski/android_notification_example/raw/master/images/entry_screenshot.png" alt="tost notification example screenshot" width="200px;"/>
<img src="https://github.com/joninvski/android_notification_example/raw/master/images/notification_screenshot.png" alt="notification area screenshot" width="200px;"/>

Compile
-------

    # Optional
    ANDROID_HOME=/home/.../android/sdk; export ANDROID_HOME

    ./gradlew compileDebug

Test
----

    # Make sure emulator is running or connected to real device
    ./gradlew connectedInstrumentTest


Code highlights
---------------

Toast notification:

    Toast.makeText(getApplicationContext(), R.string.greeting, Toast.LENGTH_SHORT).show();

Broadcast receiver to receive intente when download ends:

    mRefreshReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            // Let sender know that the Intent was received
            // by setting result code to RESULT_OK
            if (intent.getAction().equals(DATA_REFRESHED_ACTION)) {
                setResultCode(RESULT_OK);
            }
        }
    };
    registerReceiver(mRefreshReceiver, intentFilter);

Sends an ordered broadcast:

		// Creates a new BroadcastReceiver to receive a result indicating the state of MainActivity

		// The Action for this broadcast Intent is MainActivity.DATA_REFRESHED_ACTION
		// The result Activity.RESULT_OK, indicates that MainActivity is active and
		// in the foreground.

		mApplicationContext.sendOrderedBroadcast(new Intent(MainActivity.DATA_REFRESHED_ACTION), null,
				new BroadcastReceiver() {

					final String failMsg = "Download has failed. Please retry Later.";
					final String successMsg = "Download completed successfully.";

					@Override
					public void onReceive(Context context, Intent intent) {
						if ( getResultCode() != Activity.RESULT_OK) {

							// TODO:  If so, create a PendingIntent using the restartMainActivityIntent and set its flags
							// to FLAG_UPDATE_CURRENT

							final PendingIntent pendingIntent = PendingIntent
									.getActivity(context, 0,
											restartMainActivtyIntent,
											PendingIntent.FLAG_UPDATE_CURRENT);
                            createNotification()
						}
					}
				},
				null,
				0,
				null,
				null);
	}

Notification Area notification:

    // Uses R.layout.custom_notification for the layout of the notification View.
    RemoteViews mContentView = new RemoteViews(mApplicationContext.getPackageName(), R.layout.custom_notification);

    // TODO: Set the notification View's text to reflect whether or the download completed successfully
    if (success) {
        mContentView.setTextViewText(R.id.text, successMsg);
    } else {
        mContentView.setTextViewText(R.id.text, failMsg);
    }

    // TODO: Use the Notification.Builder class to create the Notification. You will have to set several pieces of information.
    Notification.Builder notificationBuilder = new Notification.Builder(context)
            .setTicker("ticker text")
            .setContentIntent(pendingIntent)
            .setSmallIcon(
                    android.R.drawable.stat_sys_warning)
            .setAutoCancel(true)
            .setContent(mContentView);

    // TODO: Send the notification
    NotificationManager mNotificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
    mNotificationManager.notify(MY_NOTIFICATION_ID, notificationBuilder.build());
