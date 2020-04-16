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

![windows variable](https://iosimage.s3.amazonaws.com/2020/74-open-specific-app-push-ios13/whereiswindow.png)



But the function that will be called when push notification is tapped is still in App Delegate, because push notification tap is app-wide, which is not limited to just one scene, how can we change the view controllers being shown inside the App Delegate code?



The main goal is to access the **window.rootViewController** from AppDelegate so we can change the structure of current view controllers presented to user.



This is how we can access the root view controller from the first scene (assuming your app has only one scene, if your app has multiple scene, you need to write code to decide which scene to use).



```swift
(UIApplication.shared.connectedScenes.first?.delegate as? SceneDelegate)?.window?.rootViewController 
```



**UIApplication.shared.connectedScenes** is a set (unordered array) that represents the current scenes of the app shown on the iOS device.



We then get the first scene using **.first** , and get its **delegate** (UISceneDelegate protocol type). 



In order to access the **window** property as defined in SceneDelegate.swift file, we need to cast it to SceneDelegate using **as? SceneDelegate** .



 Lastly, we access the rootViewController through the window property, viola!



You can then change the root view controller in the AppDelegate.swift like this : 

```swift
// when user tap on the push notification
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        
    guard var rootViewController = (UIApplication.shared.connectedScenes.first?.delegate as? SceneDelegate)?.window?.rootViewController else {
        return
    }

    // change your view controller here
    rootViewController = UIViewController()
}
```



## Example with tab bar controllers and navigation controllers

Say your app flow is like this, a tab bar controller as the initial view controller and each tab is embed inside navigation controller : 



![view flow](https://iosimage.s3.amazonaws.com/2020/74-open-specific-app-push-ios13/viewflow.png)



The root view controller is our entry point to access view controllers from AppDelegate. One naive approach would be changing the root view controller to the view controller we want to show, when the user tap on the push notification.



Say we want to show the ConversationViewController when user tap on push notification :

![show conversation view controller](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/storyboardIDVC.png)



We can change the root view controller like this :

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
      
		// the root view controller
		guard var rootViewController = (UIApplication.shared.connectedScenes.first?.delegate as? SceneDelegate)?.window?.rootViewController else {
        return
    }
    
    let storyboard = UIStoryboard(name: "Main", bundle: nil)

    // instantiate the view controller from storyboard
    if  let conversationVC = storyboard.instantiateViewController(withIdentifier: "ConversationViewController") as? ConversationViewController {

        // set the view controller as root
        rootViewController = conversationVC
    }
    
    // tell the app that we have finished processing the user’s action / response
    completionHandler()
}
```

<br>

![change root view controller directly](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/dontworry.gif)



The problem with this approach is that by replacing the root view controller with the new view controller, we have removed the whole current stack including tab bar controller and also the navigation controller, making the user unable to go back to the previous view controller! 😱



A better solution would to be pushing the new view controller into the current navigation controller (assuming every tab has its own navigation controller).



## Pushing new view controller into the current navigation controller

A better approach would be pushing the new view controller to the existing navigation controller.



This section assume that your app uses a **navigation controller** to move around view controllers. All tabs in the tab bar controller contain a navigation controller.



![storyboard](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/storyboard.png)



As the tab bar controller is the root view controller, we can traverse to the navigation controller, and push the new view controller into it like this :



```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    
    // retrieve the root view controller (which is a tab bar controller)
    guard let rootViewController = (UIApplication.shared.connectedScenes.first?.delegate as? SceneDelegate)?.window?.rootViewController else {
        return
    }
  
    let storyboard = UIStoryboard(name: "Main", bundle: nil)

    // instantiate the view controller we want to show from storyboard
    // root view controller is tab bar controller
    // the selected tab is a navigation controller
    // then we push the new view controller to it
    if  let conversationVC = storyboard.instantiateViewController(withIdentifier: "ConversationViewController") as? ConversationViewController,
        let tabBarController = rootViewController as? UITabBarController,
        let navController = tabBarController.selectedViewController as? UINavigationController {

            // we can modify variable of the new view controller using notification data
            // (eg: title of notification)
            conversationVC.senderDisplayName = response.notification.request.content.title
            // you can access custom data of the push notification by using userInfo property
            // response.notification.request.content.userInfo
            navController.pushViewController(conversationVC, animated: true)
    }
    
    // tell the app that we have finished processing the user’s action / response
    completionHandler()
}
```



This code will produce the following result :

![tap and move to correct VC](https://iosimage.s3.amazonaws.com/2019/50-open-specific-app-push/tapAndMove.gif)



--CTA--



## Presenting view controller modally

What if the current showing view controller of your app isn't contained inside a navigation controller? You can choose to present it modally (remember to check if there is any other view controller is being presented before presenting it), but you would have to traverse from the root view controller to the current showing view controller manually in AppDelegate, and then call **.present()** on the current view controller.



```swift
if  let conversationVC = storyboard.instantiateViewController(withIdentifier: "ConversationViewController") as? ConversationViewController,
    let tabBarVC = rootViewController as? UITabBarController {
    
    tabBarVC.selectedViewController.present(conversationVC, animated: true, completion: nil)
}
```

