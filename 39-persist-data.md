# When to use UserDefaults, Keychain and Core Data



There are many ways to store data locally in iOS app. UserDefaults, Keychain and Core Data are some of the most popular ways to persist data (so that the data is still there the next time user launch your app after quitting).



## UserDefaults

As per [Apple Documentation](), UserDefaults is 

> An interface to the user’s defaults database, where you store key-value pairs persistently across launches of your app.



What type of data can we store in UserDefaults? 

> A default object must be a property list—that is, an instance of (or for collections, a combination of instances of) [`NSData`](https://developer.apple.com/documentation/foundation/nsdata), [`NSString`](https://developer.apple.com/documentation/foundation/nsstring), [`NSNumber`](https://developer.apple.com/documentation/foundation/nsnumber), [`NSDate`](https://developer.apple.com/documentation/foundation/nsdate), [`NSArray`](https://developer.apple.com/documentation/foundation/nsarray), or [`NSDictionary`](https://developer.apple.com/documentation/foundation/nsdictionary). If you want to store any other type of object, you should typically archive it to create an instance of NSData.



Wait... what is a property list? 🤔 What does "A default object must be a property list" means? 



You might have seen a .plist file before, plist is short for property list. There is usually an **info.plist** file created for you when you start new iOS project :

![Info Plist](https://iosimage.s3.amazonaws.com/2018/39-persist-data/infoplist.png)



When you store data in UserDefaults, the data format is similar to Info.plist as well. The UserDefaults plist is saved in the **Library** folder inside the app folder ([Read more on app folder structure here](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html#//apple_ref/doc/uid/TP40010672-CH2-SW12)). We can take a peek into the Library folder like this : 

```swift
UserDefaults.standard.set("https://fluffy.es", forKey: "homepage")
UserDefaults.standard.set(false, forKey: "darkmode")

let library_path = NSSearchPathForDirectoriesInDomains(.libraryDirectory, .userDomainMask, true)[0]

print("library path is \(library_path)")
```

<br>

Build and run the app in Simulator, then open Finder and press <kbd>command</kbd> + <kbd>shift</kbd> + <kbd>G</kbd> , paste in the library path and click 'Go' to navigate to the Library folder.



![Library Path](https://iosimage.s3.amazonaws.com/2018/39-persist-data/libraryPath.png)



You will see two folder, 'Caches' and 'Preferences' , the UserDefaults plist file is stored inside the 'Preferences' folder.



![Library folders](https://iosimage.s3.amazonaws.com/2018/39-persist-data/libraryFolders.png)



![plist](https://iosimage.s3.amazonaws.com/2018/39-persist-data/defaultPlist.png)



If we double click the .plist file, Xcode will show a property list format to us : 

![content of plist](https://iosimage.s3.amazonaws.com/2018/39-persist-data/plistContent.png)



As UserDefaults uses .plist format to save data, we can only store data with type of NSString, NSNumber, NSData, NSArray, NSDictionary or NSData. If we want to store custom object into UserDefaults, we can use [Codable and PropertyListEncoder](https://fluffy.es/saving-custom-object-into-userdefaults/) to turn the custom object into NSData.



In the documentation, Apple mentioned few example use cases : 

> For example, you can allow users to specify their preferred units of measurement or media playback speed. Apps store these preferences by assigning values to a set of parameters in a user’s defaults database.



You can use UserDefaults for storing user settings (eg: settings page in your app with UISwitch, Segmented Control or simple Textfield)



![Settings](https://iosimage.s3.amazonaws.com/2018/39-persist-data/settingsView.png)




You can also store non-sensitive data such as high score for a game, recently played song etc. In my public transport app [Komuter](https://komuter.app), the last 5 trips are stored in UserDefaults (Array of custom objects encoded into NSData). User can then tap the recent trips when they select station.


![recent trip](https://iosimage.s3.amazonaws.com/2018/39-persist-data/recentTrip.jpg)


Avoid storing large amount of data in a single UserDefaults key such as 50 rows of user's favorite songs. 

And also **avoid storing image data** (conversion of UIImage to NSData) into UserDefaults, as UserDefaults are not meant to store large amount of data. A better way to do this is to save the image file (eg: avatar.png) into the Library folder of app, then store the path to the image (eg: "AppFolder/Library/avatar.png") into UserDefaults, and show the image using `UIImage(contentsOfFile: savedPath)`.



Storing large amount of data into UserDefaults could affect performance of your app significantly as the whole UserDefaults plist file is loaded into memory when your app launches. As mentioned in Apple Documentation : 

> `UserDefaults` caches the information to avoid having to open the user’s defaults database each time you need a default value.





## Keychain

Previously, we have explained that UserDefaults saves data into plist. Using apps such as [iExplorer](https://macroplant.com/iexplorer), users can access the Library/Preferences folder of their iPhone and read / modify the UserDefaults plist data easily (eg: Change the boolean value of "boughtProVersion" from false to true, or change the amount of coins). **Don't ever store a boolean for checking if user has bought in-app purchase in UserDefaults**! User can change it very easily (without jailbreaking) and get your goodies for free! 😬



Other than in-app purchase status, you shouldn't store user password / API Keys in UserDefaults for the same reason as well.



This is where Keychain comes in, from [Apple documentation](https://developer.apple.com/documentation/security/keychain_services) : 

> The keychain services API helps you solve this problem by giving your app a mechanism to store small bits of user data in an encrypted database called a keychain. When you securely remember the password for them, you free the user to choose a complicated one.



![keychain](https://iosimage.s3.amazonaws.com/2018/39-persist-data/keychain.png)

Most of the Keychain services API provided by Apple are written in C language and require some configuration to use 😅. To simplify the usage of keychain, we can use some open source Keychain wrapper library like [Keychain Access](https://github.com/kishikawakatsumi/KeychainAccess) . 



Using Keychain Access library, we can save / load password like this : 

```swift
// Save the user password into keychain
let keychain = Keychain(service: "com.yourcompany.yourappbundlename")
keychain["user_password"] = "correcthorsebatterystaple"

// Load the user password
let keychain = Keychain(service: "com.yourcompany.yourappbundlename")
let user_password = keychain["user_password"]
```

<br>

Look easy isn't it? The Keychain Access library has done a lot of under the hood operation for us. 😬



You should always use Keychain to store sensitive data like password, keys, certificates etc.



Data saved in Keychain can be accessed by multiple apps, provided that the data are created from the apps from the same developer. This is how SSO (Secure sign on, like you login in one app and then another app will auto login for you) in iOS app works.



## Core Data

Core Data is a huge topic in iOS Development, some developers love it, some hate it, but nevertheless it provides a lot of feature on saving/loading/using data. From [Apple documentation](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/index.html), 



> Core Data is a framework that you use to manage the model layer objects in your application. It provides generalized and automated solutions to common tasks associated with object life cycle and object graph management, including persistence.



From [Dave DeLong 's blog post on Core Data](https://davedelong.com/blog/2018/05/09/the-laws-of-core-data/) : 

>  Core Data is an “object graph and persistence framework”, which is basically like a fancy kind of [object-relational mapping](https://en.wikipedia.org/wiki/Object-relational_mapping). That means it is a whole bunch of code to help you maintain a graph (ie, a “network” of related pieces of data with a defined organization) of objects and then persist them in some fashion.



I would like to emphasize that **Core Data is not the database** nor it consists of table of rows and columns. Core Data is a framework for managing object relations, and it can save the data in 4 formats : 

1. SQLite file
2. XML file
3. Binary file
4. In-memory (RAM)



An example of object-relational mapping : Order class can have many items (Item class), meaning an Order can have multiple items. The "Order have many Item" is the **relationship** between Order class and Item class.



![core data sample](https://iosimage.s3.amazonaws.com/2018/39-persist-data/coreData.png)



```swift
// context of the persistent container of core data (where data is saved)
let context = appDelegate.persistentContainer.viewContext

// create Item of Macbook Air and Mac Mini
let macbookAir = NSEntityDescription.insertNewObject(forEntityName: "Item", into: context) as! Item
macbookAir.name = "Macbook Air"
macbookAir.price = NSDecimalNumber(decimal: 1199.00)
        
let macMini = NSEntityDescription.insertNewObject(forEntityName: "Item", into: context) as! Item
macMini.name = "Mac Mini"
macMini.price = NSDecimalNumber(decimal: 799.00)


// create Order with items Macbook Air and Mac Mini
let order = NSEntityDescription.insertNewObject(forEntityName: "Order", into: context) as! Order
    
order.items = [macbookAir, macMini]

// save the order and its items , so it persist the next time the app is opened
do {
    try context.save()
} catch let error as NSError {
    print("Could not save. \(error), \(error.userInfo)")
}
```

<br>



Aside from saving / loading object with relationships, Core Data also offer querying function which we can use to filter the data we want to load.



For example,  we can query for overdue orders (due date is earlier than today's date) using NSPredicate on Core Data :

```swift
let fetchRequest = NSFetchRequest<Person>(entityName: "Order")
// get overdue orders, ie. dueDate is earlier than current date
fetchRequest.predicate = NSPredicate(format: "dueDate < %@", Date())
do {
  overdueOrders = try managedContext.fetch(fetchRequest)
} catch let error as NSError {
  print("Could not fetch. \(error), \(error.userInfo)")
}
```

<br>



I have built a reference site on using [NSPredicate query](https://nspredicate.xyz) here if you are interested. I recommend reading Apple's own [Getting started with Core Data guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/index.html) on beginning Core Data.



Core Data is great for when you have a long list of data (eg: to-do list, list of bookmarks etc) to save / load. Especially if your data have some relationships (eg: Order with multiple items) , require custom query / filtering (eg: Getting items below certain price) or require sorting function (sort the retrieved data by price), Core Data can handles these for you out of the box.




## Comparison Summary



















