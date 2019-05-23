# How to change paid app to free app with in-app purchase, and grandfathering previous app customers using receipt





## Installing OpenSSL Pod

We will be using OpenSSL library to perform decryption of the receipt file, we will be using cocoapods to install the 'OpenSSL' pod.



If you haven't install cocoapods yet, [follow this guide to install it](https://cocoapods.org). Then after installation, open terminal, and navigate to your project folder root, and run `pod init` to generate a Podfile.



As the [default OpenSSL pod](https://github.com/FredericJacobs/OpenSSL-Pod) has not been updated for years, it didn't adapt to the new cocoapods-downloader behaviour and [will fail to install](https://github.com/FredericJacobs/OpenSSL-Pod/issues/49) if we have used it.



We will be using a forked version of the OpenSSL pod which includes the fix for cocoapods-downloader, in your Podfile, type in this line : 

```Podfile
target 'receiptz' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for receiptz
  
  # use charlesmchen's fork of OpenSSL pod
  pod 'OpenSSL', :git => 'https://github.com/charlesmchen/OpenSSL-Pod.git'
end

```

<br>



Then type and run `pod install` in the terminal, it will install and compile OpenSSL for your project, this process takes about 10 minutes on my internet connection speed. (You will need to open the **.xcworkspace** file instead of .xcodeproj file next time)



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
```

<br>



By importing these header files (pkcs7.h, objects.h) , we can then use the functions in these files on our Swift project.