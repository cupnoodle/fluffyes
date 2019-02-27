# How to debug app which got launched by tapping push notification

There might be occassion that your app got launched by user tapping push notification in the control center. Sometimes we might want to do a certain action if the app is launched from push notification, say move to a certain view controller or call a certain API, which is different than when the app receive push notification while in the foreground.



You might run into issue like API is not being called when the app is launched from push notification, no problem we can put a few **print()** statements inside the function and see what went wrong…. oh wait how do we attach a debugger / console immediately after the app is being launched from push notification?! 😱



Normally when we want to debug an app, we will open Xcode, build and run the app and attach a debugger to it. But in this case we want to attach a debugger to the app in phone **after** the app is launched by tapping notification.



Fortunately, we can actually configure Xcode to attach the debugger right after the app is launched by user on the phone. To do this, on the top menu of Xcode, select **Product** > **Scheme** > **Manage Schemes** ...



![manage schemes](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/manageScheme.png)



Then double click your app scheme name :

![enter scheme](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/enterScheme.png)



On the left sidebar, select "**Run**", then on the right panel, select "**Info**", and check "**Wait for executable to be launched**"



![wait](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/wait.png)

