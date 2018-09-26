# How to perform action when user tap on push notification (foreground and background)



> How to make the app open a specific view controller when user tap on push notification? like when you tap the notification in twitter app, it brings you to the specific tweet



I came across this problem a while ago, as many answers in Stack Overflow used Objective-C and the deprecated *didReceiveRemoteNotification* function, I thought of writing this post to use Swift and the latest UNUserNotificationCenter delegate.



Apple has deprecated the **didReceive** and **didReceiveRemoteNotification** in iOS 10,

```swift
// DEPRECATED
func application(_ application: UIApplication, didReceive notification: UILocalNotification) {
}
    
// DEPRECATED
func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any]) {
}
```

<br>



It has since been replaced with the **UNUserNotificationCenter** class after iOS 10. To handle push notification in App Delegate, we will set the delegate of UNUserNotificationCenter to self (App Delegate).

```swift
import UIKit
import UserNotifications

@UIApplicationMain

class AppDelegate: UIResponder, UIApplicationDelegate {

  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.

    UNUserNotificationCenter.current().delegate = self
    
    // request permission from user to send notification
    UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound], completionHandler: { authorized, error in
      if authorized {
        DispatchQueue.main.async(execute: {
            application.registerForRemoteNotifications()
        })
      }
    })

    return true
  }
}                    
```

<br>



And implement these two methods of **UNUserNotificationCenterDelegate** : 

```swift
extension AppDelegate: UNUserNotificationCenterDelegate{

  // This function will be called when the app receive notification
  func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
      
    // show the notification alert (banner), and with sound
    completionHandler([.alert, .sound])
  }
    
  // This function will be called right after user tap on the notification
  func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
      
    // tell the app that we have finished processing the user’s action / response
    completionHandler()
  }
}
```

<br>





