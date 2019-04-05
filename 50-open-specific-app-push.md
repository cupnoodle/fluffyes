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



Now that we can process the notification tap, how do we access the view controllers from AppDelegate?



## Window and RootViewController

AppDelegate has a **window** property that contains all the view controllers that is being displayed. 

From [Apple's](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html) [Documentation](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/CreatingWindows/CreatingWindows.html#//apple_ref/doc/uid/TP40009503-CH4-SW1),  

> A window is an instance of the `UIWindow` class and handles the overall presentation of your application’s user interface. Windows work with views (and their owning view controllers) to manage interactions with, and changes to, the visible view hierarchy



> Every iOS application needs at least one window—an instance of the `UIWindow` class—and some may include more than one window. A window object has several responsibilities:
>
> - It contains your application’s visible content.
> - It plays a key role in the delivery of touch events to your views and other application objects. 
> - It works with your application’s view controllers to facilitate orientation changes.



You can think of UIWindow as a container that contains your app views (and their owning view controllers), view controllers are usually stored in a stack (think of it like layers in Photoshop).



![UIWindow layers](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/uiwindowlayers.png)





If we click on the debug hierachy button in Xcode while the app is running, we can see the 3D arrangement of the view controllers and views. Notice that UIWindow is on the furthest back in the hierachy, the tab bar controller is stacked on top of it, and then the navigation controller, and finally the specific view controller.

![uiwindow](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/uiwindow.png)



We can access the window property (UIWindow object) in AppDelegate by using **self.window?** , self refers to the AppDelegate, and the window property is optional, it might be nil in case there is no window, but very unlikely.



There's a **rootViewController** property for the window object, which access the most bottom view controllers in the stack. In the illustration above, the most bottom view controller is the UITabBarController, this is the rootViewController for the window.

![root view controller](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/rootViewC.png)



In storyboard, the flow looks like this: 

