# How to debug app which got launched by tapping push notification

There might be occassion that your app got launched by user tapping push notification in the control center. Sometimes we might want to [do a certain action if the app is launched from push notification](https://fluffy.es/perform-action-notification-tap/), say move to a certain view controller or call a certain API, which is different than when the app receive push notification while in the foreground.



You might run into issue like API is not being called when the app is launched from push notification, no problem we can put a few **print()** statements inside the function and see what went wrong…. oh wait how do we attach a debugger / console immediately after the app is being launched from push notification?! 😱



Normally when we want to debug an app, we will open Xcode, build and run the app and attach a debugger to it. But in this case we want to attach a debugger to the app in phone **after** the app is launched by manually tapping notification.



Fortunately, we can configure Xcode to attach the debugger right after the app is launched by user on the phone. To do this, on the top menu of Xcode, select **Product** > **Scheme** > **Manage Schemes** ...



![manage schemes](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/manageScheme.png)



Then double click your app scheme name :

![enter scheme](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/enterScheme.png)



On the left sidebar, select "**Run**", then on the right panel, select "**Info**", and check "**Wait for executable to be launched**"



![wait](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/wait.png)



Now when we press the ▶️ button on Xcode, we will see that the Xcode is waiting to attach : 

![waiting to attach](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/waitingToAttach.png)



Seems good! When we launch the app from phone manually, the text will change to "Running on device".



After attaching the debugger to the app, you will notice that the **print()** statements doesn't output anything to the Xcode console, what happened? 😱  When we launch an app without attaching a debugger, the app will record the log in the device instead of outputting it to Xcode console. In this case, the output of **print()** statements are saved inside the iOS device logs, we can access it by plugging the device to Mac, open Xcode > **Windows** > **Devices and Simulators**.



![windows devices](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/devices.png)



![logs](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/devicelogs.png)



## Workaround for printing log to Xcode console

Checking device logs manually can be troublesome as it might have tons of littered logs inside.



As the log will be printed to the device log instead of Xcode console, we can use breakpoints as a workaround to print stuff on the Xcode console.



Click on the line number that will be executed to make a breakpoint, when the code execution passes through that line, the breakpoint will be triggered. 

![breakpoint](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/breakpoint.png)



After creating the breakpoint, right click on it and select **Edit breakpoint…** ,



Select **Log Message** for the action, and type in the message you want to log in the console. 

Tick the 'Automatically continue after evaluating actions' so the code execution will continue as normal instead of being paused.

![edit breakpoint](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/editbreakpoint.png)



Now when we press the ▶️ and manually open the app on the device, the log appears! 🙌



![success](https://iosimage.s3.amazonaws.com/2019/48-debug-app-launched-by-push/success.png)



Now we can debug what went wrong in the didFinishLaunchingWithOptions method.



