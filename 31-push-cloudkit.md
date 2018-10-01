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
4. Your iOS device is logged in with a valid iCloud account



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

The way CloudKit subscription work is that the app will subscribe to its CloudKit container's record type. Whenever there's a new record created in CloudKit Dashboard, Apple's Cloud server would send a push notification to the app. It's like subscribing to my blog email list and you will get email whenever a new post is posted, you won't need to keep checking my blog for new post, 😉.



You can read the [official Apple guide on CloudKit](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/SubscribingtoRecordChanges/SubscribingtoRecordChanges.html) here.



The code to subscribe for CloudKit is as follow : 

```swift
// AppDelegate.swift

// When user allowed push notification and the app has gotten the device token
    // (device token is a unique ID that Apple server use to determine which device to send push notification to)
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        
        // Create a subscription to the 'Notifications' Record Type in CloudKit
        // User will receive a push notification when a new record is created in CloudKit
        // Read more on https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/SubscribingtoRecordChanges/SubscribingtoRecordChanges.html
        
        // The predicate lets you define condition of the subscription, eg: only be notified of change if the newly created notification start with "A"
        // the TRUEPREDICATE means any new Notifications record created will be notified
        let subscription = CKQuerySubscription(recordType: "Notifications", predicate: NSPredicate(format: "TRUEPREDICATE"), options: .firesOnRecordCreation)
        
        // Here we customize the notification message
        let info = CKNotificationInfo()
        
        // this will use the 'title' field in the Record type 'notifications' as the title of the push notification
        info.titleLocalizationKey = "%1$@"
        info.titleLocalizationArgs = ["title"]
        
        // if you want to use multiple field combined for the title of push notification
        // info.titleLocalizationKey = "%1$@ %2$@" // if want to add more, the format will be "%3$@", "%4$@" and so on
        // info.titleLocalizationArgs = ["title", "subtitle"]
        
        // this will use the 'content' field in the Record type 'notifications' as the content of the push notification
        info.alertLocalizationKey = "%1$@"
        info.alertLocalizationArgs = ["content"]
        
        // increment the red number count on the top right corner of app icon
        info.shouldBadge = true
        
        // use system default notification sound
        info.soundName = "default"
        
        subscription.notificationInfo = info
        
        // Save the subscription to Public Database in Cloudkit
        CKContainer.default().publicCloudDatabase.save(subscription, completionHandler: { subscription, error in
            if error == nil {
                // Subscription saved successfully
            } else {
                // Error occurred
            }
        })
        
    }
```

<br>



We create a **CKQuerySubscription** object to subscribe for the record type "Notifications" (which we created in previous step), **.firesOnRecordCreation** means that a new push notification will be sent when a new record is created, you can also use .firesOnRecordUpdate which means that a new push notification will be sent whenever an existing record data is upated. The **predicate** parameter is a condition we can set for the subscription, let's say if we only want to be notified if the record's **content** field contain the word "promotion", we can use the predicate like this : *NSPredicate(format: "content CONTAINS %@", "Promotion")* . 



Then we can customize the message of the notification that will be sent using the **CKNotificationInfo()** object.



The **titleLocalizationKey** and **titleLocalizationArgs** might seem foreign at first, it is used to customize the text that will appear on the notification title.



Like in Swift String , we can put a variable in the print statement like this :

```swift
let name = "Steve Jobs"
print("Hello \(name)")
// outputs 'Hello Steve Jobs'
```

<br>



The **%1$@** in the **titleLocalizationKey** is similar to above, which it will be replaced by the text in the **title** field of the Notifications record type.



![key arguments](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/keyArgs.png)



**alertLocalizationKey** and **alertLocalizationArgs** works the same too, except that alertLocalizationKey / Args will be used for the content text of the push notification instead of title.



Then we include this customized notification data to the CKQuerySubscription : 
`subscription.notificationInfo = info` . This notification data will be used whenever our subscription detects new data.



Build and run the app on your physical devices (not simulator). Then go to the CloudKit Dashboard, click the  'Subscriptions', select 'Public Database' and click 'Fetch Subscriptions' , you should see a new subscription created. This is the subscription created by the app.



![Subscription row](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/subscriptionRow.png)





## Step 5 - Send the push notification!

In the CloudKit Dashboard (app container's development data), create a new Record for the '**Notifications**' type, remember to select '**Public Database**' in the Database field. Type in your desired text into the 'content', 'title' and 'subtitle' field, then click 'Save'.

 

![Create a New Record](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/newRecord.png)



You should see a push notification appear on your iPhone / iPad now 🎉 :

![push notification banner](https://iosimage.s3.amazonaws.com/2018/31-push-cloudkit/banner.png)



Easy right? No need to deal with certificates, provisioning profiles, device tokens and even Firebase.

You can send push notification just by creating a new record in CloudKit Dashboard!



## What if I want to let user choose not to receive notification in the app?

If user want to opt out of notification, you can cancel their subscription by putting the following code in maybe settings view controller of you app : 




// cancel subscription in app

ibaction 

loop through subscription and remove them



// example conditional subscribing

// like only receive notification if the user is male/female etc

