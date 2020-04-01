# Inspect app folder in simulators and real device

Your app might store some data or files in the Documents folder, and sometimes you might want to check if the data / files are stored correctly by inspecting them. How do we find the app folder containing the data?





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