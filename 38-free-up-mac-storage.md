# Free up storage space of your Mac for iOS development

It's easy to run out of disk space when you are doing iOS development on your Mac, especially if your Mac has only 128GB of storage. 



[TL;DR Jump to DevCleaner app](#devcleanerapp)



Ever gotten this warning when you want to update Xcode? I hope not! 😱



![App store space](https://iosimage.s3.amazonaws.com/2018/38-free-up-mac-storage/xcodespace.jpg)



## iOS Simulators

Usually the main culprit taking up space will be the **iOS Simulators**. As you download newer version of iOS Simulator incrementally (iOS 10, 11 , then 12), the space add up as the old simulators file still remain.



The Simulator files are usually located in **~/Library/Developer/Xcode/iOS DeviceSupport/**. 



![device support](https://iosimage.s3.amazonaws.com/2018/38-free-up-mac-storage/deviceSupport.png)



Open Finder, press <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>G</kbd> , paste in `~/Library/Developer/Xcode/iOS DeviceSupport/`  and press Enter. You should see multiple folder containing different iOS version of simulators. You can delete unused older version of simulator here. These files **can take up to dozens of GBs**, be sure to check these every time a new iOS version is introduced.



![Quick go](https://iosimage.s3.amazonaws.com/2018/38-free-up-mac-storage/quickgo.png)







## App Archive

The second culprit will be **App Archives**. These are created every time you archive an app, these can build up slowly as you archive different version of your apps.



To delete these archive data in Xcode, select **Window** > **Organizer** > **Archives** from top menu. Then press <kbd>delete</kbd> to remove them.



![archive](https://iosimage.s3.amazonaws.com/2018/38-free-up-mac-storage/archives.png)



App archives can take up significant space if your app is large in size, a rough calculation would be taking your app size and multiply it by the number of time you have archived.



## Derived data 

Last but not least, **derived data** can take up significant space too. Derived data is generated during each app build process. Derived data contains intermediate build results, generated indexes, that help speed up build time.



You can think of derived data as cached data/image in web browser, most of the time web browser will auto-save the data/image into your hard disk when you first browse a web page, and the next time you visit the same web page, the pageload speed is faster as web browser will use the cached data /image in hard disk. 



Similar to iOS Simulator, derived data is located in **~/Library/Developer/Xcode/DerivedData** , you can safely delete them. To delete derived data of the current open Xcode project, you can hold <kbd>alt</kbd>, click **Product** > **Clean Build Folder**.



![derived data](https://iosimage.s3.amazonaws.com/2018/38-free-up-mac-storage/derivedData.png)



<span id="devcleanerapp"></span>

## DevCleaner ⚡️

It can be a hassle to clean up simulators, archive data and derived data manually once in a while. 

Fortunately, there's a [free Mac app](https://itunes.apple.com/us/app/devcleaner/id1388020431?mt=12) by Konrad which can handle these at ease.

[Dev Cleaner](https://itunes.apple.com/us/app/devcleaner/id1388020431?mt=12) allow us to select which data to remove easily : 



![dev cleaner](https://iosimage.s3.amazonaws.com/2018/38-free-up-mac-storage/devcleaner.png)



I have been using DevCleaner for a while and really like its simplicity, kudos to Konran for making this app.





