# Toggle iCloud sync on/off for NSPersistentCloudKitContainer

I wanted an option for user to toggle iCloud sync on / off for my own app [AuthCat](https://authcat.app) (2FA OTP app with iCloud sync).

![icloud sync toggle](https://iosimage.s3.amazonaws.com/2020/81-toggle-icloud-sync/icloudsync.png)



Some users may not feel comfortable sharing their data to cloud, or just prefer to not sync data between devices, so it is important to have an option to let them disable it. My app default is iCloud sync off, they have to turn it on manually.



There's some NSPersistentCloudKitContainer tutorial online, but there isn't much guide on how to implement the sync toggling functionality, so I have decided to write this.



This tutorial assume that you already have a NSPersistentCloudKitContainer ready in your app. You can check [Andrew's excellent tutorial](https://www.andrewcbancroft.com/blog/ios-development/data-persistence/getting-started-with-nspersistentcloudkitcontainer/) on how to implement NSPersistentCloudKitContainer if you haven't already implement it.



## A boolean to store if the sync is on/off

First, you will need to have a boolean variable to store whether the user has turned the sync on or off, and this variable must be accessible by other iOS devices owned by the same Apple ID as well.



To make this boolean variable accessible on another iOS device, we need to store it using **iCloud Key-Value store** (NSUbiquitousKeyValueStore). You can think of it like UserDefaults, but the value is shared across different devices for the same app, that ties to an Apple ID account.



Enable the iCloud key-value storage capability in your project :

![icloud key value enable](https://iosimage.s3.amazonaws.com/2020/81-toggle-icloud-sync/enablekeyvalue.png)



Then in the UISwitch's IBAction, update the boolean value when user toggle the switch :

```swift
@IBAction func iCloudToggled(_ sender: UISwitch) {
    // set the icloud_sync key to be true/false depending on the UISwitch state
    NSUbiquitousKeyValueStore.default.set(sender.isOn, forKey: "icloud_sync")
}
```



Similar to UserDefaults, if the value for the key is not set previously, it will return false by default.



I have used the key name "icloud_sync" here, you can use other key name if you want to.



## Disable cloud sync on NSPersistentCloudKitContainer

When you initialize a NSPersistentCloudKitContainer stack, by default it will sync to an iCloud container (the remote database), with the container identifier you specified in the capabilities section : 

![containers name](https://iosimage.s3.amazonaws.com/2020/81-toggle-icloud-sync/containers.png)



If you print out the default value of the container identifier like this, you will get the same identifier shown in the capabilities section : 

```swift
// your CoreData stack in AppDelegate.swift

lazy var persistentContainer: NSPersistentCloudKitContainer = {
  
  let container = NSPersistentCloudKitContainer(name: "AppName")

  guard let description = container.persistentStoreDescriptions.first else {
      fatalError("###\(#function): Failed to retrieve a persistent store description.")
  }
  
  // this will output "iCloud.es.fluffy.AuthCat"
  print("cloudkit container identifier : \(description.cloudKitContainerOptions?.containerIdentifier)")
  
  // ....
}
```



To disable iCloud syncing, we can **set the cloudKitContainerOptions property to nil**. By setting it to nil, the NSPersistentCloudKitContainer will not connect to any cloud container, hence no sync will occur.



We can then decide whether to set this property to nil based on the boolean we set earlier. Note that we need to set the cloudKitContainerOptions property to nil before we load the persistentStore.



```swift
// AppDelegate.swift
lazy var persistentContainer: NSPersistentCloudKitContainer = {
  let container = NSPersistentCloudKitContainer(name: "AppName")

  guard let description = container.persistentStoreDescriptions.first else {
      fatalError("###\(#function): Failed to retrieve a persistent store description.")
  }

  description.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)

  description.setOption(true as NSNumber,
                              forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)

  // if "icloud_sync" boolean key isn't set or isn't set to true, don't sync to iCloud
  if(!NSUbiquitousKeyValueStore.default.bool(forKey: "icloud_sync")){
      description.cloudKitContainerOptions = nil
  }

  container.loadPersistentStores(completionHandler: { (storeDescription, error) in
  })
  // ...
}
```



We have set "true" for the NSPersistentHistoryTrackingKey, so that when the user decide to turn on iCloud sync back, all the Core Data changes made during off-sync is tracked and synced to iCloud.



## Reinitialize the persistent container when iCloud sync is toggled





## Delete existing data from iCloud when iCloud sync is turned off





