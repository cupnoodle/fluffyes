# How to change paid app to free app with in-app purchase, and grandfathering previous app customers using receipt

App Store has since become a race to bottom for developers in term of pricing their app, it can be very hard to convince a user to pay for an app before they even download it. Most apps in the App Store now has switched into the business model of free with in-app purchase for additional feature, this allows the user to try out the basic app function before buying the premium feature, and also increases conversion rate.



Say you already have a paid app in App Store and want to convert it into free with in-app purchase, but doing so might cause an upset of customers who have already bought it previously, as they will need to fork out money to buy the in-app purchase again. How can we check if an user has previously purchased and downloaded the paid app? and also unlock in-app purchase if the user has previously bought the app?



We can check if an user has previously downloaded the app by checking the download receipt, there's a "original app version" property on the receipt which indicates the version number of the app that the user first downloaded using their Apple account.



## App Store receipt

When a user downloads app from the App Store, App Store will generate an app receipt and bundle it with the app.

![app bundle with receipt](https://iosimage.s3.amazonaws.com/2019/55-paid-to-free/appdownload.png)



We can access the receipt file using **Bundle.main.appStoreReceiptURL** ,

// testflight / dev



## Installing OpenSSL Pod

We will be using OpenSSL library to perform decryption of the receipt file, we will be using cocoapods to install the 'OpenSSL' pod.



If you haven't install cocoapods yet, [follow this guide to install it](https://cocoapods.org). Then after installation, open terminal, and navigate to your project folder root, and run `pod init` to generate a Podfile.



We will be using the [OpenSSL-Universal](https://github.com/krzyzanowskim/OpenSSL) pod for installing the OpenSSL library.

```Podfile
target 'receiptz' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for receiptz
  
  pod 'OpenSSL-Universal'
end

```

<br>



Then type and run `pod install` in the terminal, it will install and compile OpenSSL for your project. (You will need to open the **.xcworkspace** file instead of .xcodeproj file next time)



As OpenSSL is written in C,  we will to need to create an Objective-C bridging header to access the functionality provided in the OpenSSL library.



If you haven't create an Objective-C bridging header yet, it's easy to add one as Xcode will auto create one for us if we create a new Objective-C File. Follow these steps :



Right-click on your project folder in Xcode, select **New File…** , then select "**Objective-C File**"  and click Next.



You can enter any file name for this file, as we only want to trigger the Xcode auto bridging header generator with this, we will delete the objective-C file later.



After creating the Objective-C file, Xcode will prompt "Would you like to configure an Objective-C bridging header", click "**Create Bridging Header**".



A bridging header file will be created, inside the file, we will insert the following import statements :

```c
// yourAppName-Bridging-Header.h
//
//  Use this file to import your target's public headers that you would like to expose to Swift.
//

#import <openssl/pkcs7.h>
#import <openssl/objects.h>
#import <openssl/sha.h>
#import <openssl/x509.h>
```

<br>



By importing these header files (pkcs7.h, objects.h, etc) , we can then use the functions in these files on our Swift project. For the receipt validation, these functions will be used for decryption purpose.