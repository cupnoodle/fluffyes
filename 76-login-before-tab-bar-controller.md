# How to implement login screen with main tab bar controller

This article assumes your app will only have one scene all of the time. (ie. no two scenes at the same time in iPad).



At the end of this post, you will be able to implement a login screen (a navigation controller which can push to registration view), that transition into the main tab controller like this : 



![demo achievement](https://iosimage.s3.amazonaws.com/2020/76-login-before-tab-bar-controller/demo.gif)

















// Pain

// if check user login status on tab bar controller, and push

// tab bar controller shows briefly before showing login screen



// and if user keep login/logout (unlikely but still), the views keep stacking on top each other, adding memory usage



// root controller explanation (graphic of layers)





// here's how the storyboard looks like



// use userdefaults to check if user is logged in





// instead of using push, we switch the root view controller directly



// on app launch, we check if user is logged in, and show the correct view controller

