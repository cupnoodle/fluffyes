# Inspect app folder in simulators and real device

Your app might store some data or files in the Documents folder, and sometimes you might want to check if the data / files are stored correctly by inspecting them. How do we find the app folder containing the data?



You can also [inspect the UserDefaults data](https://fluffy.es/persist-data/#userdefaults) by going to **AppFolder/Library/Preferences/Your_app_bundle_id.plist** .



## Simulators

For simulator, the app folder is stored in your Mac. You can get the path of this folder by printing  **NSHomeDirectory()**



```swift
print("app folder path is \(NSHomeDirectory())")
```

<br>



You can put this line in viewDidLoad function, and get the path in the Xcode console.

![path](https://iosimage.s3.amazonaws.com/2020/73-inspect-app-folder/folderPath.png)



Copy the path, open Finder, then select "**Go**" -> "**Go to Folder**" (shortcut key Command + Shift + G), then paste in the path and press Enter.



![go to folder](https://iosimage.s3.amazonaws.com/2020/73-inspect-app-folder/gotofolder.png)



![paste path](https://iosimage.s3.amazonaws.com/2020/73-inspect-app-folder/pastePath.png)



![app folder](https://iosimage.s3.amazonaws.com/2020/73-inspect-app-folder/appFolder.png)





## Real device

For real device, you will need to download the whole app container to your Mac to inspect them.



Open Xcode, Select  **Window** > **Devices and Simulators**

![devices and simulators](https://iosimage.s3.amazonaws.com/2020/73-inspect-app-folder/devices.png)



Select your device, then choose the app you want to inspect, then select "**Download container**" to download the app folder to your Mac.

![download container](https://iosimage.s3.amazonaws.com/2020/73-inspect-app-folder/downloadContainer.png)



The app container will be in .xcappdata format, you can inspect it by right clicking it -> "**Show package contents**"

![show package](https://iosimage.s3.amazonaws.com/2020/73-inspect-app-folder/showPackage.png)



![app folder](https://iosimage.s3.amazonaws.com/2020/73-inspect-app-folder/yay.png)



