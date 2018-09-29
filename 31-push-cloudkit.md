# Set up Push Notification easily using CloudKit



> How do I generate certificates for push notification?

> Device tokens? APNS? Environment?!

> Is there a way to send push notification without relying on a backend? Firebase? 

> Why the Firebase notification doesn't appear on my device?!



A small task of wanting to send push notification can quickly turn into a nightmare of certificates, device tokens, troubleshooting why push notification works in development environment but not on Testflight etc.

Firebase does ease the pain a bit, but you will need to install the huge Firebase SDK library, still need to generate certificates or keys to use it and requires additional step to debug when push notification doesn't arrive.



"I just want to send a generic push notification to all of my users"

The good news is that Apple's own CloudKit can handle these for you (for free too!), you wouldn't need to worry about certificates / keys / device tokens / APNS if you use CloudKit.



This tutorial assume that

1. Your app doesn't rely on a existing backend server to send push notification.
2. You are enrolled in Apple Developer Program, or have access to one
3. You have a physical iOS device (push notification won't work in Simulator)



Table of Contents:




## Step 1  - Enabling Push Notifications and CloudKit capabilities in your app

In your Xcode Project, select a valid team for Signing if you haven't already :

![Change Team](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/changeTeam.png)



Enable **Push Notification** in the App Target > Capabilities

![Capabilities](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/capabilities.png)



![push notification](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/push.png)



Enable **iCloud** in the App Target > Capabilities, and check **CloudKit** 
![iCloud](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/iCloud.png)



Next, we will create a record type in CloudKit to store the data for the notification.



## Step 2 - Create record types in CloudKit

This post won't go into detail on explaining CloudKit, but here's an overview of what is RecordType, Record and Field mean (similar to a spreadsheet) :

![Cloudkit description](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/cloudkitDescription.png)



Click the **Cloudkit Dashboard** button shown previously to go to the Apple Cloudkit Dashboard, or go to [https://icloud.developer.apple.com/dashboard](https://icloud.developer.apple.com/dashboard) .



Open the container for your app,

![Container](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/appContainer.png)



Select Development > Data,

![Development](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/devData.png)



Development Data are separated with Production Data, Development will be used when you are developing the app in your mac / phone. Production will be used when your app is in the App Store.



Select **Record Types**, Create a new type named **Notifications**. And add fields named **content**, **subtitle** and **title**. Click Save.



![Record type](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/recordType.png)



Next, we will add some code to our app so that the app will get notified (by push notification) when a new record is created in the CloudKit dashboard.



## Step 3 - Add Push notification handling code

Add the code below in **AppDelegate.swift** 's didFinishLaunchingWithOptions method to request permission from user to send notifications:

```swift
// AppDelegate.swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
  // Override point for customization after application launch.
  
  // set self (AppDelegate) to handle notification
  UNUserNotificationCenter.current().delegate = self

  // Request permission from user to send notification
  UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound], completionHandler: { authorized, error in
    if authorized {
      DispatchQueue.main.async(execute: {
        application.registerForRemoteNotifications()
      })
    }
  })
  
  return true
}
```

<br>



And add the following code to make AppDelegate conform to the UNUserNotificationCenterDelegate protocol.



```swift
extension AppDelegate: UNUserNotificationCenterDelegate{
    
  // This function will be called when the app receive notification
  func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    
    // show the notification alert (banner), and with sound
    completionHandler([.alert, .sound])
  }
  
  // This function will be called right after user tap on the notification
  func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    
    // tell the app that we have finished processing the user’s action (eg: tap on notification banner) / response
    completionHandler()
  }
}
```

<br>



## Step 4 - Add CloudKit Subscription code 



 





// cancel subscription in app

ibaction 

loop through subscription and remove them



// example conditional subscribing

// like only receive notification if the user is male/female etc

