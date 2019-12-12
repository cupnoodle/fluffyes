# Sign in With Apple button implementation tutorial

In WWDC 2019, Apple has announced a new privacy-focused mechanism for user to sign in on third party app, named "**Sign in with Apple**" (SiwA). If your app currently allows user to sign in with third party providers such as Facebook, Google, Twitter etc, you will also need to add a "Sign in with Apple" option for user, failure to comply might result in rejection during App Review.



According to this [Apple guide](https://developer.apple.com/news/?id=09122019b), existing apps in App Store has a deadline of April 2020 to implement SiwA, and new app submitted to App Store (after September 2019) must implement SiwA if the app supports other third party logins.

> Starting today, new apps submitted to the App Store must follow these guidelines. Existing apps and app updates must follow them by April 2020.



And from [App Store review guideline](https://developer.apple.com/app-store/review/guidelines/#sign-in-with-apple), your app **does not need to implement SiwA** if your app exclusively uses your company own login system.

> Sign in with Apple is not required if:
>
> - Your app exclusively uses your company’s own account setup and sign-in systems.
> - Your app is an education, enterprise, or business app that requires the user to sign in with an existing education or enterprise account.
> - Your app uses a government or industry-backed citizen identification system or electronic ID to authenticate users.
> - Your app is a client for a specific third-party service and users are required to sign in to their mail, social media, or other third-party account directly to access their content.





// table of contents



## Prerequisite

To implement Sign in With Apple, you **must have a paid Apple developer account**. As the Sign in With Apple button is a custom UIControl, we can't create it in Storyboard / XIB, we would have to create it in code  and add constraints to it programmatically, I recommend reading a bit on [how to create UI / constraint programmatically](https://fluffy.es/intro-to-creating-ui-in-code-2/) if you are not farmiliar with it yet.



## Enable Sign-in with Apple in App Identifier

If you are creating a new app, I recommend going to [Apple developer center](https://developer.apple.com/account/) and create a new app identifier  with Sign-in With Apple capability enabled.



Head over to [Apple developer center](https://developer.apple.com/account/), select "Certificates, Identifier and Profiles", 

![Certificates, Identifier and Profiles](https://iosimage.s3.amazonaws.com/2019/67-siwa/cip.png)



Select "Identifier" and click "+" to add a new identifier, then select "App ID".

![identifiers](https://iosimage.s3.amazonaws.com/2019/67-siwa/identifiers.png)



![app identifier](https://iosimage.s3.amazonaws.com/2019/67-siwa/appID.png)



Ensure the bundle ID is **explicit**, and use the same Bundle ID in the Xcode project : 

![explicit bundle identifier](https://iosimage.s3.amazonaws.com/2019/67-siwa/explicitBundleID.png)

![bundle identifier](https://iosimage.s3.amazonaws.com/2019/67-siwa/XcodeBundleIdentifier.png)



In the "Register an App ID" page, scroll down and find "Sign-in with Apple" capability, then check the box : 
![enable SIWA](https://iosimage.s3.amazonaws.com/2019/67-siwa/enableSIWA.png)



Click "Continue" to create the App ID.



If you already have an existing app, select the corresponding App Identifier in the Identifiers list, and check the "Sign-in with Apple" capability and click "Save".



## Enable Sign-in with Apple capability in Xcode

Next, open up your Xcode project, select your project in the left side bar, then in the "Signing & Capabilities" tab, click "+" and select "Sign in with Apple".



![capabilities](https://iosimage.s3.amazonaws.com/2019/67-siwa/capabilityXcode.png)



![sign in with apple capability](https://iosimage.s3.amazonaws.com/2019/67-siwa/siwaCap.png)



This capability would allow your app to use the "Sign-in with Apple" feature.



## Setting up SIWA button UI

To use "Sign-in with Apple" function, you have to use the specially designated **ASAuthorizationAppleIDButton** class, you can't use your own custom button or risk rejection by App Review team.



Unfortunately there's no storyboard / XIB option for this button, we have to add it programmatically to our view controller / view.



We have to import the **AuthenticationServices** before using SIWA :



```swift
import AuthenticationServices

class ViewController: UIViewController {
  //...
}
```



Then in viewDidLoad() function, add the sign-in with Apple button : 

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.

    let siwaButton = ASAuthorizationAppleIDButton()

    // set this so the button will use auto layout constraint
    siwaButton.translatesAutoresizingMaskIntoConstraints = false

    // add the button to the view controller root view
    self.view.addSubview(siwaButton)

    // set constraint
    NSLayoutConstraint.activate([
        siwaButton.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor, constant: 50.0),
        siwaButton.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor, constant: -50.0),
        siwaButton.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -70.0),
        siwaButton.heightAnchor.constraint(equalToConstant: 50.0)
    ])

    // the function that will be executed when user tap the button
    siwaButton.addTarget(self, action: #selector(appleSignInTapped), for: .touchUpInside)
}

// this is the function that will be executed when user tap the button
@objc func appleSignInTapped() {
}
```

<br>

![sign in button](https://iosimage.s3.amazonaws.com/2019/67-siwa/buttonLayout.png)



Looking good! When user tap on the button, the **appleSignInTapped()** method will be executed. Currently it doesn't do anything, we will implement this function next.



## Setting up Sign-in with Apple dialog

We want to show the Sign-in with Apple system dialog when user tap on the button, which is the **ASAuthorizationController** . 



![ASAuthorizationController](https://iosimage.s3.amazonaws.com/2019/67-siwa/ASAuthorizationController.png)



To show this dialog, we need to supply **ASAuthorizationAppleIDRequest** to the initializer of the ASAuthorizationController, which can be obtained using **ASAuthorizationAppleIDProvider**.



```swift
@objc func appleSignInTapped() {
    let provider = ASAuthorizationAppleIDProvider()
    let request = provider.createRequest()
    // request full name and email from the user's Apple ID
    request.requestedScopes = [.fullName, .email]

    // pass the request to the initializer of the controller
    let authController = ASAuthorizationController(authorizationRequests: [request])
  
    // delegate functions will be called when user data is
    // successfully retrieved or error occured
    authController.delegate = self
  
    // similar to delegate, this will ask the view controller
    // which window to present the ASAuthorizationController
    authController.presentationContextProvider = self
    
    // show the Sign-in with Apple dialog
    authController.performRequests()
}
```

<br>



If we build and run the app, we will get an error, because we haven't implemented the **delegate** and **presentationContextProvider** protocol yet.















