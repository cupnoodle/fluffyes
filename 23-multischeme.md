# Use multiple target / scheme for different server environment URL

Sometimes when you are working on an iOS app that connects to server, there might be multiple server environment like development, staging and production. (God bless you if you only have one server which is the production 😅) These servers has different URL, say the development server URL is dev.example.com, and the production server URL is example.com . How do you separate these URL in your app?

Do you comment and uncomment different URLs to switch environment like this?
```swift
struct Endpoints {
    // development server URL
    static let baseURL = "https://fluffa.herokuapp.com/"
    
    // production server URL
    // static let baseURL = "https://randomfox.ca/"

    static func randomFox() -> URL {
        return URL(string: Endpoints.baseURL + "floof")!
    }
}
```
<br>

While this approach is straightforward, it is very prone to human mistake, what if you forgot to switch back to production server when archiving? Ever shipped an app which connects to development server to the App Store? (I have, and I am definitely not proud about it)

In this post we will use target / scheme in Xcode for separating different server environment. The technique shown in this post is only applicable for Swift and Xcode 8+.

This post assume that you already knew what is [URLSession](https://fluffy.es/nsurlsession-urlsession-tutorial/).

Table of contents :
1. [Duplicate current target in project](#duplicate)
2. [Add custom flag to the development target app](#customflag)
3. [Using flag and preprocessor to switch server URL](#preprocessor)
4. [Use different app icon for different target](#appicon)
5. [Caveats](#caveats)
6. [Notes](#notes)

<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> Download the sample project to follow along the post</h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/xproj.png" style="max-width: 150px;"></img>Get sample Xcode project which uses two scheme / target, one for development and another for production server
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/253108311/submissions" method="post" data-drip-embedded-form="253108311">
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
                <input type="submit" value="Send me the project file!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips to help you become a better iOS developer.<br> No Spam. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>

<span id="duplicate"></span>
## Duplicate current target in project
First, we will go to the Project Setting, select your current App Target, right-click and select "**Duplicate**".

![Duplicate Target](https://iosimage.s3.amazonaws.com/2018/23-multischeme/duplicateTarget.png)

After clicking **Duplicate**, you will see another target has been created, with a "copy" at the end of its name. We can use this newly created target for development server environment. Let's double click it to rename the target name to "(AppName) Development". 

![Rename and change bundle identifier](https://iosimage.s3.amazonaws.com/2018/23-multischeme/rename.png)

We will also change the **Display name** so the app will have a different name on the iPhone then we can differentiate the development and production app.

We will also change the bundle identifier to another name. This is because no two apps can share the same bundle identifier in the same device. If two apps share the same bundle identifier, the iOS device / simulator will remove the first app and replace it with the second app if you install the second app in the same device.

When we duplicate a target, Xcode will create a new scheme for us as well, the newly created scheme will use the new target we created by default.

We can change the scheme like this: 
![Change scheme](https://iosimage.s3.amazonaws.com/2018/23-multischeme/selectScheme.png)

You can change the scheme and build the app. After building both scheme, you will see two apps appear in the simulator. (eg: *MultiScheme* and *MultiScheme Development*)

![Two scheme apps](https://iosimage.s3.amazonaws.com/2018/23-multischeme/twoScheme.png)

Next, we will proceed to rename the scheme so that it will share the same name as target. Similar as previous step, click the scheme beside build button and select "Manage Scheme".

![Manage Scheme](https://iosimage.s3.amazonaws.com/2018/23-multischeme/manageScheme.png)

Then click on the "(appName) copy" and press **Enter** to rename it. We will rename it to "(appName development) to match the Target name.

![Rename Scheme](https://iosimage.s3.amazonaws.com/2018/23-multischeme/renameScheme.png)

<span id="customflag"></span>
## Add custom flag to the development target app

Go to project settings, select the development target, and follow the instruction below to add a custom flag to the development target: 
![Add custom flag to target](https://iosimage.s3.amazonaws.com/2018/23-multischeme/setFlag.png)

Type in "DEVELOPMENT" for the flag and press Enter. Now you will have a "DEVELOPMENT" flag when you build and run the development target. Do the same for the "Release" row, so both "Debug" and "Release" row will have "DEVELOPMENT" flag.

![Both flag](https://iosimage.s3.amazonaws.com/2018/23-multischeme/releaseFlag.png)


When you start a new iOS project, Xcode will create two build configuration, **Debug** and **Release**, Xcode will also add a "DEBUG" flag into the Debug build configuration by default. 

What are these build configuration used for?

In the "Manage Scheme" Window, double click into the scheme of your app, and you will see a list of actions on the left side (eg: Build, Run, Test etc).

Select **Run** on the left side and you will see "Debug" is selected for the "Build Configuration" by default.
![Run Scheme](https://iosimage.s3.amazonaws.com/2018/23-multischeme/runScheme.png)

This mean that if you build and run the app in simulator / your plugged iPhone, the app will use the "Debug" build configuration, and it will have the "DEBUG" flag. Remember we have also added the "DEVELOPMENT" flag previously in the Debug row for the development target app? If we build and run this app, the app will have both "DEBUG" and "DEVELOPMENT" flag.

Now select **Archive** on the left side, and you will see "Release" is selected for the "Build Configuration". This mean that when we archive the app for App Store submission, the app will use the "Release" build configuration. 

If we didn't put "DEVELOPMENT" flag into the Release row earlier, it won't appear in the app if we archive this app and submit to App Store / Testflight (which might make the app use production server instead of development).

![Archive Scheme](https://iosimage.s3.amazonaws.com/2018/23-multischeme/archiveScheme.png)

<span id="preprocessor"></span>
## Using flag and preprocessor to switch server URL

Now we gotten the flag ready, we can use it to switch between server URL when a different target is selected.
In your networking/endpoint code, add the `#IF DEVELOPMENT` preprocessor check like this :
```swift
struct Endpoints {
    #if DEVELOPMENT
    static let baseURL = "https://fluffa.herokuapp.com/"
    #else
    static let baseURL = "https://randomfox.ca/"
    #endif
	
    static func randomFox() -> URL {
        return URL(string: Endpoints.baseURL + "floof")!
    }
}
```

<br>

Code with `#` (youngster calls it hashtag?) in front are known as preprocessor. What preprocessor does is that it will process the source code before it is compiled into an app. The `#if` ,`#else` and `#endif` preprocessor will check if a flag is defined for the app or not during preprocessing stage and modify the source code based on the condition. The flow looks like this: 

![Preprocessor flow](https://iosimage.s3.amazonaws.com/2018/23-multischeme/preprocess.png)

Notice that during preprocessing stage, the preprocessor will check the flag to use different line of baseURL. So now you have it! You can change the target to development during development to use the dev server, and archive the production target for App store submission.

<span id="appicon"></span>
## Use different app icon for different target
Now we have two app target for development and production, but they share the same icon which can be confusing sometimes. Let's use another app icon for the development target.

Go to `Assets.xcassets` , click the "+" below and select "App Icons & Launch Images" > "New iOS App Icon". Put in your icon, then rename it.

Then go to project settings, select the development target app, go to "Build Settings", search for "app icon" and replace the value of **Asset Catalog App Icon Set Name** with the name you used for the App Icon earlier.

![App Icon Dev](https://iosimage.s3.amazonaws.com/2018/23-multischeme/appIconDev.png)

Build the development target app and you will see it uses another set of App Icon, much easier to differentiate between development and production app now : 
![Two Icons](https://iosimage.s3.amazonaws.com/2018/23-multischeme/twoIcons.png)

<span id="caveats"></span>
## Caveats
Using multiple targets is handy for multi server situation, but it is not without caveat / downside. Here are some stuff you might need to pay attention to once you use a multi target strategy.

### Targets selection when creating new files
When you create a new file by right clicking on Xcode, notice that in the dialog box there is a "Targets" at the bottom. Remember to choose all of the targets (eg: Development and Production) so that this new file / code will work on these targets. You might get compilation error if you missed the target.

![File targets](https://iosimage.s3.amazonaws.com/2018/23-multischeme/newFileTargets.png)

### Cocoapods installation on multiple targets
If you are using Cocoapods and want to install the same pods into multiple targets, you can modify the Podfile to this:  
```ruby
# Podfile
 
platform :ios, '11.0'
 
use_frameworks!
 
def app_pods
    pod 'Alamofire'
    pod 'SDWebImage'
end
 
target 'MultiScheme' do
    app_pods
end
 
target 'MultiScheme Development' do
    app_pods
end
```

<br>

You can read more about this on [Natasha The Robot blog](https://www.natashatherobot.com/cocoapods-installing-same-pod-multiple-targets/).

### Different plist for different target

When you create a new target, Xcode will create a new plist file and assign it to the new target automatically. If you want to make adjustments on plist file (eg: App Transport Security etc), you would need to adjust both plist.

You can change the plist in the Build Settings of the target : 
![Plist](https://iosimage.s3.amazonaws.com/2018/23-multischeme/plist.png)

<span id="notes"></span>
## Notes
Put down your pitchfork if you have it in your hand, I know some developers [oppose the idea of using preprocessor](https://qualitycoding.org/xcode-preprocessor-macros/) saying it is bad practice, I understand the concern that the code you see might not be the exact code that will be compiled due to preprocessing and such. I have used this strategy and worked with other developers who use this strategy as well, it works well if you take care of its caveats 🤔.

<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> Download the sample project to follow along the post</h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/xproj.png" style="max-width: 150px;"></img>Get sample Xcode project which uses two scheme / target, one for development and another for production server
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/253108311/submissions" method="post" data-drip-embedded-form="253108311">
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
                <input type="submit" value="Send me the project file!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips to help you become a better iOS developer.<br> No Spam. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>