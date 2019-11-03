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

