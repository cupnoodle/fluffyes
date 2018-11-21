# Free up storage space of your Mac for iOS development

It's easy to run out of disk space when you are doing iOS development on your Mac, especially if your Mac has only 128GB of storage. 



Ever gotten this warning when you want to update Xcode? I hope not! 😱



![App store space](https://iosimage.s3.amazonaws.com/2018/38-free-up-mac-storage/xcodespace.jpg)



## iOS Simulators

Usually the main culprit taking up space will be the **iOS Simulators**. As you download newer version of iOS Simulator incrementally (iOS 10, 11 , then 12), the space add up as the old simulators file still remain.



The Simulator files are usually located in **~/Library/Developer/Xcode/iOS DeviceSupport/**. 

Open Finder, press <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>G</kbd> , paste in `~/Library/Developer/Xcode/iOS DeviceSupport/`  and press Enter. You should see multiple folder containing different iOS version of simulators. You can delete unused older version of simulator here. These files **can take up to dozens of GBs**, be sure to check these every time a new iOS version is introduced.



## App Archive

The second culprit will be **App Archives**. These are created every time you archive an app, these can build up slowly as you archive different version of your apps.



To delete these archive data in Xcode, select **Window** > **Organizer** > **Archives** from top menu. Then press <kbd>delete</kbd> to remove them.



App archives can take up significant space if your app is large in size, a rough calculation would be taking your app size and multiply it by the number of time you have archived.



## Derived data 

Last but not least, **derived data** can take up significant space too. Derived data is generated during each app build process, t









