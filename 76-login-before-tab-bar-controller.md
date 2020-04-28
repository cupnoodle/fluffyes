# How to implement login screen with main tab bar controller

This article assumes your app will only have one scene all of the time. (ie. no two scenes at the same time in iPad).



At the end of this post, you will be able to implement a login screen (a navigation controller which can push to registration view), that transition into the main tab controller like this : 



![demo achievement](https://iosimage.s3.amazonaws.com/2020/76-login-before-tab-bar-controller/demo.gif)





Here are some common mistakes I saw other developers did (I did it too when first starting out) : 

## Common mistakes

### 1. Presenting the main view from login view on viewDidAppear every time

You might have attempted to present or push the main view controller on the login view controller when user has logged in previously, using code like this : 

```swift
// LoginViewController.swift

override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    
    // if user has logged in previously, present the main view controller
    
    if let loggedUsername = UserDefaults.standard.string(forKey: "username") {
        let storyboard = UIStoryboard(name: "Main", bundle: nil)
        let mainTabBarController = storyboard.instantiateViewController(identifier: "MainTabBarController")
        mainTabBarController.modalPresentationStyle = .fullScreen
        
        self.present(mainTabBarController, animated: true, completion: nil)
    }
}
```

<br>



This works but user will have to see the present/push animation every time they launch the app, if they have already logged in previously : 



![slow mo modal](https://iosimage.s3.amazonaws.com/2020/76-login-before-tab-bar-controller/slowmomodal.gif)

(I used slow animation so it is easier to nice).



### 2. Keep presenting view controllers without dismissing

When user taps login, the main view controller is pushed/presented on top, then when user logs out, the login view controller is pushed / presented on top, and keeps repeating. This might look like a seamless experience, but it will keep the view controllers stacked without ever being released from memory, which can cause the app to run out of memory and crash eventually if user keeps on login and logout (unlikely but still).



If you never dismiss the view controllers, they will keep stacking on top like this : 



![stack](https://iosimage.s3.amazonaws.com/2020/76-login-before-tab-bar-controller/stack.png)



And memory usage will keep increasing as more view controllers are added on top of the stack, notice the memory keeps increasing when I present a view controller : 



![keep pushing](https://iosimage.s3.amazonaws.com/2020/76-login-before-tab-bar-controller/keeppushing.gif)





## Changing root view controller

Instead of pushing/ presenting view controller endlessly, a better way to do this is to change the root view controller directly. **Root view controller is the bottom-most view controller** on the stack. If you are using storyboard, the root view controller is the initial view controller you have set in your storyboard :



![initial view controller](https://iosimage.s3.amazonaws.com/2020/76-login-before-tab-bar-controller/initialVC.png)



Instead of presenting the main tab controller on top of login view controller like this : 

![present](https://iosimage.s3.amazonaws.com/2020/76-login-before-tab-bar-controller/present.png)





We are going to switch the bottom-most view controller (root view controller) from the login view to main tab view like this : 

![switch root](https://iosimage.s3.amazonaws.com/2020/76-login-before-tab-bar-controller/switchroot.png)



















// Pain

// if check user login status on tab bar controller, and push

// tab bar controller shows briefly before showing login screen



// and if user keep login/logout (unlikely but still), the views keep stacking on top each other, adding memory usage



// root controller explanation (graphic of layers)





// here's how the storyboard looks like



// use userdefaults to check if user is logged in





// instead of using push, we switch the root view controller directly



// on app launch, we check if user is logged in, and show the correct view controller

