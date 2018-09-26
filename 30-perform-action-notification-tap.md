# How to perform action when user tap on push notification (foreground and background)



> How to make the app open a specific view controller when user tap on push notification? like when you tap the notification in twitter app, it brings you to the specific tweet



[TL;DR: Jump to the final code](#answer) 



![push tapped gif](https://iosimage.s3.amazonaws.com/2018/30-perform-action-notification-tap/pushtapped.gif)



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



Remember to execute the **completionHandler** for both of the delegate methods as specified by Apple in their documentation ( [willPresent notification:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate/1649518-usernotificationcenter), [didReceive response:](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate/1649501-usernotificationcenter)).



Now we can use the **didReceive response:** to detect when user tap on the notification. To differentiate between when user tapped the notification while the app is running in foreground and when the app is in background, we can use **UIApplication.shared.applicationState** to check whether the app is in foreground or background when user tapped the notification.



<span id="answer"></span>

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
  let application = UIApplication.shared
  
  if(application.applicationState == .active){
    print("user tapped the notification bar when the app is in foreground")
    
  }
  
  if(application.applicationState == .inactive)
  {
    print("user tapped the notification bar when the app is in background")
  }
  
  /* Change root view controller to a specific viewcontroller */
  // let storyboard = UIStoryboard(name: "Main", bundle: nil)
  // let vc = storyboard.instantiateViewController(withIdentifier: "ViewControllerStoryboardID") as? ViewController
  // self.window?.rootViewController = vc
  
  completionHandler()
}
```

<br>



> How do I perform action when user launch the app by tapping on notification?

If the app is not running / not in background and user tap on the notification , the app will be launched. The method **didFinishLaunchingWithOptions launchOptions:** will be called and iOS will add the **remoteNotification** key to the launchOptions.



You can check the launchOptions for remoteNotification key to perform action when user launch the app from tapping on notification : 

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
  // Override point for customization after application launch.
  
  // When the app launch after user tap on notification (originally was not running / not in background)
  if(launchOptions?[UIApplicationLaunchOptionsKey.remoteNotification] != nil){
      
  }
  
  return true
}
```

<br>





### Further reading

Keith Harrison has wrote a great post on [using local notifications](https://useyourloaf.com/blog/local-notifications-with-ios-10/) if you would like to learn more about UNUserNotificationCenter.







