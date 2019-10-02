# Breaking changes checklist before building your app using iOS 13 SDK 

iOS 13 does bring a lot of exciting feature like Dark mode, Multi-tasking, Sign in with Apple etc. Other than exciting feature, iOS 13 SDK does bring some breaking changes as well, overlooking these changes might cause your app to behave differently as expected 😅. This article will go through some breaking changes I have found and workaround for it.



## Modal presentation

If you have used modal presentation for view controller prior to iOS 13, the default behavior is presenting the modal view controller in full screen, and when the modal view controller is dismissed, the **viewDidAppear** function will be called on the presenting view controller (ie. the view controller that is responsible for presenting the modal view controller).



However in iOS 13, the default behavior of presenting the modal view controller is replaced with a card-like animation (the official term for it is **page sheet**) :



![card](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/modal.png)



The card modal view in iOS 13 has shorter height than the full screen modal in iOS 12 (and previous iOS versions).

![shorter height](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/modalComparison.png)



By default, user can swipe down the presented card modal view in iOS 13 to dismiss it. This may cause issue if your application flow is to dismiss the modal programmatically only after user has completed certain actions.



And after the dismissal, the **viewDidAppear** function will **not** be called on the presenting view controller (the view controller that presents the modal), if you are doing UI changes in this function, those changes might not be implemented!



### Sticking to previous full screen modal behaviour

If you want to preserve the old full screen modal behaviour, you can set the modal presentation style to "full-screen" (previous iOS versions before), instead of using the default.



To set it programmatically, set the modally presented view controller's **.modalPresentationStyle** property to **.fullscreen** .

```swift
@IBAction func presentModalTapped(_ sender: Any) {
    let detailVC = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(identifier: "DetailViewController")
    
    // use back the old iOS 12 modal full screen style
    detailVC.modalPresentationStyle = .fullScreen
        
    self.present(detailVC, animated: true, completion: nil)
}
```

<br>



If you are doing the presentation using segue in Storyboard, select the segue, and change it to "fullscreen"

![storyboard modal full screen](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/storyboardModal.png)





### Preventing user to dismiss

Say you want to adopt the new card-like modal presentation style, but don't want user to be able to swipe down to dismiss, simple set the modal's **isModalInPresentation** to **true** (default is false).



```swift
// the VC being presented modally
// disallow user to swipe down to dismiss

if #available(iOS 13.0, *) {
  // this property is available for iOS 13 and above only
  detailVC.isModalInPresentation = true
}
```



[Apple Documentation](https://developer.apple.com/documentation/uikit/uiviewcontroller/3229894-ismodalinpresentation) :

> The default value of this property is [`false`](https://developer.apple.com/documentation/swift/false). When you set it to [`true`](https://developer.apple.com/documentation/swift/true), UIKit ignores events outside the view controller's bounds and prevents the interactive dismissal of the view controller while it is onscreen.



![cant dismiss](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/unableToDismiss.gif)



### New delegate method for presentation controller

If you use the new modal presentation style and wants to execute code when user dismiss the modal, you can't rely on the "viewDidAppear" function on the presenting view controller. Fortunately, Apple has added a few new method to the protocol **UIAdaptivePresentationControllerDelegate**.



![presentation delegate](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/presentationDelegate.png)



![modal flow delegate methods](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/modalFlow.png)



Sarun has written an excellent guide on [explaining how to use these delegate functions here](https://sarunw.com/posts/modality-changes-in-ios13/),  I recommend reading his guide on this matter.





## Large title navigation bar appearance

Prior to iOS 13, you might have used **UINavigationBar.appearance()** to change the color / font of the navigation bar like this : 

```swift
 // button color
UINavigationBar.appearance().tintColor = .white

// title color
UINavigationBar.appearance().titleTextAttributes =
    [NSAttributedString.Key.foregroundColor: UIColor.white]

// large title color
UINavigationBar.appearance().largeTitleTextAttributes =
    [NSAttributedString.Key.foregroundColor: UIColor.white]

// background color
UINavigationBar.appearance().barTintColor = .blue
```

<br>



If you used the same code to build for iOS 13, you might find that your large title navigation bar doesn't follow the styling at all! 😱

![ios 13 navigation bar wtf](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/navigationbar.png)



In iOS 13, Apple has introduced a new class, **UINavigationBarAppearance**, to replace the old UINavigationBar.appearance() approach. Here's an example : 



```swift
if #available(iOS 13, *) {
  let appearance = UINavigationBarAppearance()

  // title color
  appearance.titleTextAttributes = [.foregroundColor: UIColor.white]

  // large title color
  appearance.largeTitleTextAttributes = [.foregroundColor: UIColor.white]

  // background color
  appearance.backgroundColor = .blue

  // bar button styling
  let barButtonItemApperance = UIBarButtonItemAppearance()
  barButtonItemApperance.normal.titleTextAttributes = [.foregroundColor: UIColor.white]

  appearance.backButtonAppearance = barButtonItemApperance

  // set the navigation bar appearance to the color we have set above
  UINavigationBar.appearance().standardAppearance = appearance

  // when the navigation bar has a neighbouring scroll view item (eg: scroll view, table view etc)
  // the "scrollEdgeAppearance" will be used
  // by default, scrollEdgeAppearance will have a transparent background
  UINavigationBar.appearance().scrollEdgeAppearance = appearance
}

// the back icon color
UINavigationBar.appearance().tintColor = .white
```

<br>



The new UINavigationBarAppearance has added in three mode of appearance, **standard**, **compact** and **scrollEdgeAppearance**. Each mode can have different styling / coloring.



Standard appearance is the normal navigation bar we see on portrait iPhone / iPad, compact appearance is the shorter navigation bar we see on landscape iPhone (and some other scenario). Scroll edge appearance is the appearance when a large title navigation bar is used with a scroll view right below it, and when the user haven't scroll it yet. Once user scroll the scrollview, a standard appearance will be used.



![navigation mode](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/navigationMode.png)



![navigation scroll](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/navigationScroll.png)





If you didn't specify an appearance for scrollEdgeAppearance, iOS by default will use a transparent background for it, so that you can see the content below scrolling through it. Hence you saw the blanked out navigation bar on iOS 13 earlier on.



To change the appearance of bar button in navigation bar, we can use **UIBarButtonItemAppearance()**  object, then assign it to the navigation appearance like this : 



```swift
let appearance = UINavigationBarAppearance()

let barButtonItemApperance = UIBarButtonItemAppearance()

// ... do your styling here

// appearance for both button
appearance.buttonApperance

// appearance for back button
appearance.backButtonAppearance = barButtonItemApperance

// appearance for done button
appearance.doneButtonAppearance

```

<br>



One thing to take notice is that even after setting back button appearance, the back arrow icon color doesn't get styled, only the "back" text on the back button get styled.



![back arrow color problem](https://iosimage.s3.amazonaws.com/2019/64-ios13-checklist/arrowTintColor.png)



So far the solution I have found for this is using the good 'ol **UINavigationBar.appearance().tintColor = .white** to change the color of the back button arrow.



## Push notification

### Sending push notification 
According to Apple's [documentation](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns), the header **apns-push-type** is required for sending push notification (to Apple Push Server) for WatchOS 6 or later, and is recommended for iOS, macOS app as well.



> (Required for watchOS 6 and later; recommended for macOS, iOS, tvOS, and iPadOS) The value of this header must accurately reflect the contents of your notification’s payload. If there is a mismatch, or if the header is missing on required systems, APNs may return an error, delay the delivery of the notification, or drop it altogether.



**apns-push-type** has 6 valid values (as of 2 October 2019), which indicate the type of push :

1. alert
2. background
3. voip
4. complication
5. fileprovider
6. mdm





### Push notification device token

In your AppDelegate, if you are retrieving push notification device token string using the **.description** method like this : 

```swift
// ⚠️ Please don't do this
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
  let tokenData = deviceToken as NSData
  let token = tokenData.description

  let token = "\(deviceToken)".replacingOccurrences(of: " ", with: "")
                              .replacingOccurrences(of: "<", with: "")
                              .replacingOccurrences(of: ">", with: "")
  
  // then send your token to server...
}
```

<br>



This will not work in iOS 13 as Apple has changed the output for the **.description** method for objects.

```swift
// iOS 12
(deviceToken as NSData).description // "<789dc438 6d688f2c e2dd66d8 0d335454 5ce29f0f ed71234a f6b37ddb 8a504523>"

// iOS 13
(deviceToken as NSData).description // "{length = 32, bytes = 789dc438 6d688f2c ... f6b37ddb 8a504523 }"

// the same regex method won't work on iOS 13
```

<br>



The solution for this is to read each byte of DeviceToken (as it is a NSData object, it contains sequence of bytes) and combining all the bytes.

```swift
let deviceTokenString = deviceToken.map { String(format: "%02x", $0) }.joined()
```

<br>



Matt from NSHipster has [written an excellent guide addressing this issue](https://nshipster.com/apns-device-tokens/), here's an excerpt on how the device token reading works : 

> - The `map` method operates on each element of a sequence. Because `Data` is a sequence of bytes in Swift, the passed closure is evaluated for each byte in `deviceToken`.
> - The [`String(format:)`](https://developer.apple.com/documentation/swift/string/3126742-init) initializer evaluates each byte in the data (represented by the anonymous parameter `$0`) using the [`%02x` format specifier](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Strings/Articles/formatSpecifiers.html), to produce a zero-padded, 2-digit hexadecimal representation of the byte / 8-bit integer.
> - After collecting each byte representation created by the `map` method,`joined()` concatenates each element into a single string.



If you are using external service for push notification (such as [OneSignal](https://onesignal.com/blog/ios-13-introduces-4-breaking-changes-to-notifications/) etc), it might be good idea to contact their support to check if their SDK code is using the **description** method for device token retrieval, if yes, then they would have to change it for iOS 13.



## Opting out of Dark mode



























