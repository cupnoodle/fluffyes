# Open app in specific view when push notification is tapped (iOS 13+)

Say you have an app and want to redirect user to a specific view when a push notification is tapped, eg: going to a specific chat room in Telegram after tapping push notification of that message. How do we proceed to implement this? 🤔



**This tutorial assumes your app have only one scene**.



At the end of this tutorial, you will be able to implement this : 

![tap push and move to view](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/tapAndMove.gif)





(Notice that after tapping the push notification, the app moves to the conversation with Sans).



In the previous article on [performing action when user tap on notification](https://fluffy.es/perform-action-notification-tap/), we know that to perform an action when user tap on notification, we need to use the **didReceive response:** method from UNUserNotificationCenterDelegate.

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



Now that we can process the notification tap, how do we access the view controllers from AppDelegate?



## Scene and Window

Since iOS 13, Apple has introduced a new concept called 'Scene' , as in you can open more than one scene of the same app.



![scenes](https://iosimage.s3.amazonaws.com/2020/74-open-specific-app-push-ios13/scenes.png)



And when you create a new iOS app project using Xcode 11 and above, Xcode will create a SceneDelegate.swift for you. Prior to iOS 13, the top of the view hierachy is the UIWindow, and we can access the root view controller of a window by using **window.rootViewController** in the AppDelegate. 



But since iOS 13, the top  of the view hierachy is not UIWindow anymore, it's UIScene , and the UIWindow is one level below it.

![root view VC](https://iosimage.s3.amazonaws.com/2020/74-open-specific-app-push-ios13/rootviewvc.png)



The window variable is not available anymore in AppDelegate, which is moved to SceneDelegate instead. 



But the function that will be called when push notification is tapped is still in App Delegate, because push notification tap is app-wide, which is not limited to just one scene, how can we update the view controllers being shown inside the App Delegate code?



To do this....



