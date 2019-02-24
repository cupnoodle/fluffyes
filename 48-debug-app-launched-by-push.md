# How to debug app which got launched by tapping push notification

There might be occassion that your app got launched by user tapping push notification in the control center. Sometimes we might want to do a certain action if the app is launched from push notification, say move to a certain view controller or call a certain API, which is different than when the app receive push notification while in the foreground.



You might run into issue like API is not being called when the app is launched from push notification, no problem we can put a few **print()** statements inside the function and see what went wrong…. oh wait how do we attach a debugger immediately when the app is being launched from push notification?! 😱



Normally when we want to debug an app, we will open Xcode, build and run the app and attach a debugger to it. But in this case we want to attach a debugger to the app in phone immediately 

