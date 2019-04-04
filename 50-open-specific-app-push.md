# Open app in specific view when push notification is tapped

Say you have an app and want to redirect user to a specific view when a push notification is tapped, eg: going to a specific chat room in Telegram after tapping push notification of that message. How do we proceed to implement this? 🤔



![tapAndMove](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/tapAndMove.gif)



(Notice that after tapping the push notification, the app moves to the conversation with Sans).



In this article on [performing action when user tap on notification](https://fluffy.es/perform-action-notification-tap/), we know that to perform an action when user tap on notification, we need to use the **didReceive response:** method from UNUserNotificationCenterDelegate.



```swift
// AppDelegate.swift

// remember to set delegate for notification center 
// UNUserNotificationCenter.current().delegate = self

extension AppDelegate: UNUserNotificationCenterDelegate{
    
  // This function will be called right after user tap on the notification
  func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
      
    // tell the app that we have finished processing the user’s action / response
    completionHandler()
  }
}
```

<br>







