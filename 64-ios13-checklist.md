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



In iOS 13, Apple has introduced a new class, **UINavigationBarAppearance**, to replace the old UINavigationBar.appearance() approach.















