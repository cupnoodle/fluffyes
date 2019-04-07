# Open app in specific view when push notification is tapped

Say you have an app and want to redirect user to a specific view when a push notification is tapped, eg: going to a specific chat room in Telegram after tapping push notification of that message. How do we proceed to implement this? 🤔



[In a rush and just want the code solution? Click here](#tldr)



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

![storyboard](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/storyboard.png)



The root view controller is our entry point to access view controllers from AppDelegate. One naive approach would be changing the root view controller to the view controller we want when the user tap on the push notification.



Say we want to show the ConversationViewController when user tap on push notification : 

![CVC Storyboard ID](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/storyboardIDVC.png)



We can change the root view controller like this :



```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
      
    let storyboard = UIStoryboard(name: "Main", bundle: nil)

    // instantiate the view controller from storyboard
    if  let conversationVC = storyboard.instantiateViewController(withIdentifier: "ConversationViewController") as? ConversationViewController {

        // set the view controller as root
        self.window?.rootViewController = conversationVC
    }
}
```

<br>



![dont worry gif](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/dontworry.gif)



The problem with this approach is that by replacing the root view controller with the new view controller, we have removed the whole current stack including tab bar controller and also the navigation controller, making the user unable to go back to the previous view controller! 😱



## Pushing new view controller into the current navigation controller

A better approach would be pushing the new view controller to the existing navigation controller.

This section assume that your app uses a **navigation controller** to move around view controllers. All tabs in the tab bar controller contain a navigation controller.



![storyboard](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/storyboard.png)



As the tab bar controller is the root view controller, we can traverse to the navigation controller, and push the new view controller into it like this : 

<span id="tldr"></span>

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        
    let storyboard = UIStoryboard(name: "Main", bundle: nil)

    // instantiate the view controller from storyboard
    // root view controller is tab bar controller
    // the selected tab is a navigation controller
    // then we push the new view controller to it
    if  let conversationVC = storyboard.instantiateViewController(withIdentifier: "ConversationViewController") as? ConversationViewController,
        let tabBarController = self.window?.rootViewController as? UITabBarController,
        let navController = tabBarController.selectedViewController as? UINavigationController {

            // we can modify variable of the new view controller using notification data
            // (eg: title of notification)
            conversationVC.senderDisplayName = response.notification.request.content.title

            navController.pushViewController(conversationVC, animated: true)
    }
}
```

<br>



This code will produce the following result : 

![tapAndMove](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/tapAndMove.gif)



## Presenting view controller modally

What if the current showing view controller of your app isn't contained inside a navigation controller? You can choose to present it modally (remember to check if there is any other view controller is being presented before presenting it), but you would have to traverse from the root view controller to the current showing view controller manually in AppDelegate, and then call **.present()** on the current view controller.



```swift
if  let conversationVC = storyboard.instantiateViewController(withIdentifier: "ConversationViewController") as? ConversationViewController,
    let tabBarVC = self.window?.rootViewController as? UITabBarController {
    
    tabBarVC.selectedViewController.present(conversationVC, animated: true, completion: nil)
}
```

<br>



<script async data-uid="82f4a0256c" src="https://f.convertkit.com/82f4a0256c/b60b8439b5.js"></script>