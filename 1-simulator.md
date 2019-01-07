# How to run iOS app on another Mac's iOS Simulator



## How to Archive for iOS simulator?

Usually you won't archive iOS app for simulator use, unless... your client suddenly tell you that his iPhone is broken and undergoing repair, but hey he has a Mac that can run Xcode and also iOS simulator and he still want to test your app! Of course you wouldn't want to send the full source code over before client paid you in full. So how do you send him the build to test without sending the source code over?

As per [Apple documentation](https://developer.apple.com/library/content/technotes/tn2215/_index.html), 
> Applications built for the simulator cannot be archived 


Fortunately, there's a way to send the compiled build so your client can run the compiled build without him having to build from the source code.

## Update (October 2018)
[honeyeeb](https://honeyeeb.com) mentioned a faster way that we can use the compiled app in the '**Products**' folder, send this to colleague/client and ask them to drag and drop it into iOS Simulator.

After building the app, go to the left side navigator bar, select **Products > appName.app**, right click it and select '**Show in Finder**'.

![show in finder](https://iosimage.s3.amazonaws.com/2018/1-simulator/show_in_finder.png) 

Finder will show the .app file and you can send the file to your colleague / client.

Thanks for the suggestion honeyeeb! 😆

## 1. Locate your iOS Simulator folder in Finder

Open terminal, and run `instruments -s devices` , you will see a list of simulator device and their UDID inside square brackets.  

![Simulator list](https://iosimage.s3.amazonaws.com/2018/1-simulator/find_simulator.png)

I will be using iPhone SE (iOS 11.2) as the selected simulator device for this tutorial, the corresponding UDID for it is "18BF1A2D-15C2-40E2-80A6-0CB87D2B56D4".

The folder containing the simulator data will be located at
`~/Library/Developer/CoreSimulator/Devices/[Simulator UDID]` .
In Finder, press <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>G</kbd>, and enter the path.

![Simulator data path](https://iosimage.s3.amazonaws.com/2018/1-simulator/simulator_path.png)



## 2. Build your app and locate its data folder

Before building your app, I recommend deleting all other existing app you have built previously on the simulator so that you can find your app data folder easier in the next step.

Proceed to build your app as usual in Xcode, my app name is "exampleApp" for this tutorial.

After building, open Finder and proceed to the simulator folder path
`~/Library/Developer/CoreSimulator/Devices/[Simulator UDID]`

In your simulator folder, locate to `data/Containers/Bundle/Application` , here you will see the folder for the apps you have built in the simulator.

![App data folder](https://iosimage.s3.amazonaws.com/2018/1-simulator/app_data_folder.png)

To find the folder containing your app, you have to open one by one until you found your app name inside, like this :

![Correct app folder](https://iosimage.s3.amazonaws.com/2018/1-simulator/correct_app_folder.png)

Compress the app and send the zip file to your client.

![Compress selected app](https://iosimage.s3.amazonaws.com/2018/1-simulator/compress_app.png)

For easier instruction, ask your client to unzip the compressed file at Desktop folder.

<a name="client"></a>

## 3. Instruction for clients

Below are the instruction for clients: 
Open Xcode, then start iOS Simulator by choosing Xcode > Open Developer Tool > Simulator.

![Open Simulator from Xcode](https://iosimage.s3.amazonaws.com/2018/1-simulator/open_simulator_tool.png)

In Simulator, select the device you want.

![Select simulator device](https://iosimage.s3.amazonaws.com/2018/1-simulator/simulator_select_device.png)

Simply drag and drop the app file into the Simulator : 
![Drag drop app](https://iosimage.s3.amazonaws.com/2018/1-simulator/dragDropApp.gif)

The app should install on the simulator successfully, rejoice!

Send this link to your client for reference if needed : [https://fluffy.es/how-to-archive-ios-app-for-simulator/#client](https://fluffy.es/how-to-archive-ios-app-for-simulator/#client)

<div class="post-subscribe">
  <div class="post-subscribe-left">
     <h4 data-drip-attribute="headline">Want to level up your iOS development skills?</h4>
    <span style="font-size:0.9rem;"> 
            Sign up below and I'll send you articles just like this about iOS development to your inbox every two week-ish.
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/295054774/submissions" method="post" data-drip-embedded-form="295054774">
                <div style="margin-bottom: 0.5rem;">
                    <label for="drip-firstname">Name<span style="color:#952B45;">*</span></label><br />
                    <input type="text" id="drip-firstname" name="fields[firstname]" value="" />
                </div>
                <div>
                    <label for="drip-email">Email Address<span style="color:#952B45;">*</span></label><br />
                    <input type="email" id="drip-email" name="fields[email]" value="" />
                </div>
              <div>
                <br>
                <input type="submit" value="Sign me up!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips.<br> Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>