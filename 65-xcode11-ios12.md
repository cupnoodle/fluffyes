# Allow app created in Xcode 11 to run on iOS 12 and lower

If you create a new app in Xcode 11 and try to run it on an iOS 12 device or lower, you will get a bunch of errors : 

![errors](https://iosimage.s3.amazonaws.com/2019/65-xcode11-ios12/spookyError.png)



Notice that most of these errors are related to the **UIScene** class and the **SceneDelegate.swift** file, these are related to the **multi-window feature** introduced in iOS 13 ,which allow multiple windows of an app to be opened in iPad. As iOS 12 and earlier don't have these feature, we will get these error messages when trying to compile.



We will try to resolve these errors step by step in this article.



You can save the hassle by setting deployment target to iOS 13 and above, and ditch support for iOS 12 and below 😈. But keep in mind that **half of the iOS devices** in circulation **are not** using iOS 13 as of mid October 2019, according to [Apple's report](https://developer.apple.com/support/app-store/) : 



![iOS Percentage](https://iosimage.s3.amazonaws.com/2019/65-xcode11-ios12/iOSPercentage.png)

If you are dropping support for iOS 12 and below, you are blocking half of the iOS users to use your app! 😱





## Update deployment target

If you haven't already, change the deployment target to the lowest iOS version you want to support, select the project name then select your app target, choose **General** and change the version in Deployment Info.

![change deployment target](https://iosimage.s3.amazonaws.com/2019/65-xcode11-ios12/deploymentTarget.png)



## @available out the SceneDelegate.swift

As the SceneDelegate class is only available on iOS 13 and above, we have to tell the compiler to only include the class for iOS 13 and above. To do this, we will add this line "**@available(iOS 13.0, *)**" right above the  SceneDelegate class declaration like this : 

```swift
import UIKit

@available(iOS 13.0, *)
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
//...
}
```

<br>



## @available out some methods in AppDelegate.swift

Next, there are two new methods added in AppDelegate.swift, which only supports iOS 13 and above. We will add the same @available(iOS 13.0, *) on top of them as well : 



```swift
// AppDelegate.swift

@available(iOS 13.0, *)
func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
    // Called when a new scene session is being created.
    // Use this method to select a configuration to create the new scene with.
    return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
}

@available(iOS 13.0, *)
func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
    // Called when the user discards a scene session.
    // If any sessions were discarded while the application was not running, this will be called shortly after application:didFinishLaunchingWithOptions.
    // Use this method to release any resources that were specific to the discarded scenes, as they will not return.
}
```

<br>



## Add back the window to AppDelegate

If you build and run your app now, you will get a dark black screen 😱, because there's no UIWindow initialized.



In iOS 12 and older, there's always a **var window: UIWindow?** variable located at the top of AppDelegate.swft. iOS 13 has moved this variable to SceneDelegate.swift, and now we are going to add back this variable to AppDelegate.

```swift
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
     
    var window: UIWindow?
  
    // ...
}
```

<br>



Now Build and run your app on an iOS 12 devices, and it works! 🥳



I guess Apple really wants iOS developers to adopt and focus on iOS 13, to the extent that they don't mind breaking support for iOS 12 and older with default settings in Xcode.



If you are lazy to do these step manually every time, you can also download [Xcode 10.3 in the Apple's developer download portal](https://developer.apple.com/download/more/) (require sign in with your Apple ID), create a new Xcode project using it, and then edit it using Xcode 11.