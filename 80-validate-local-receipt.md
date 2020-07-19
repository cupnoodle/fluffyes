# Read and validate in-app purchase receipt locally using TPInAppReceipt



## Disclaimer and Caveats

This tutorial is aimed to get you on feet to validate and read the in-app purchase receipt data as easy as possible using a library, this might make your app easier for jailbreaker to retrieve your app's premium content for free. 



The reason so many tutorials out there asks you to include OpenSSL library as **static library** in the bundle, is to make it harder for the hacker (user who jailbroken their device) to replace the library during run time with a hacked version to work around validation checks. For example, if the library has a function that checks for if the in-app purchase is bought ( *func isPurchased()* ), they can replace it with a hacked library that always return true for this function.



Most Stack Overflow answers and tutorials also advocate for rolling your own validation function, to make it harder for hacker to guess and modify them. If you are using an open source library on Github for receipt validation, the hacker can just simply read the source of the library and know which part they should modify to get your premium content for free.



My belief is that people who decide to pirate your app won't buy it anyway, and there's no stopping a determined hacker to disassemble your app and get your premium content for free, you can only make it harder for them to do so, but usually it is not worth the effort as the percentage of people who decide to hack your app is very small.



If you don't care about the potential piracy issue and just want a straightforward way to check if the user has purchased your in-app purchase and show the premium content to them, this tutorial is for you.



## Prerequisite

This tutorial assumes you

1. already have a paid Apple developer account
2. have created an app in App Store Connect
3. have created the in-app purchase product in App Store Connect with status 'Ready to Submit'
4. have implemented the in-app purchase flow in your app, as in you can purchase the product already



More info on [how to troubleshoot your in-app purchase setup here](https://fluffy.es/zero-iap-products-checklist/).



## Installation

We will be using the [TPInAppReceipt](https://github.com/tikhop/TPInAppReceipt) library created by Pavel for reading and validating the in-app purchase receipt. Thanks Pavel!



### Cocoapods

Add this line into your Podfile :

```swift
pod 'TPInAppReceipt'
```

<br>

then run `pod install` .



In any swift file you'd like to use TPInAppReceipt, import the framework using `import TPInAppReceipt`.



### Carthage

Add this line into your Cartfile : 

```swift
github "tikhop/TPInAppReceipt" 
```

<br>

then run `carthage update` .



## Getting the receipt data

TPInAppReceipt library encapsulates the receipts data in an **InAppReceipt **object, you can retrieve the local receipt data using **.localReceipt()** method like this : 

```swift
import InAppReceipt

if let receipt = try? InAppReceipt.localReceipt() {
  // do your validation or parsing here
} else {
  print("Receipt not found")
}
```



When a user downloads app from the App Store, App Store will generate an app receipt and bundle it with the app.



![receipt from app store](https://iosimage.s3.amazonaws.com/2019/55-paid-to-free/appdownload.png)



We can access the receipt file using **Bundle.main.appStoreReceiptURL** , the URL string might look like this : `YourAppPath/StoreKit/Receipt`.  The **InAppReceipt.localReceipt()** is a function that wraps around the Bundle.main.appStoreReceiptURL and attempt to read the file located at the URL.



When we build the app using Xcode or download it using Testflight, **usually the receipt is not included**, and the localReceipt() method will throw an error saying receipt not found.



We can manually request a receipt from the sandbox App Store by calling the StoreKit function **SKReceiptRefreshRequest.start()** . ([Read more here](https://fluffy.es/migrate-paid-app-to-iap/#appstorereceipt)) . TPInAppReceipt library also provides a wrapper around this function, we can call **InAppReceipt.refresh** to retrieve or refresh the receipt file : 

```swift
InAppReceipt.refresh { (error) in
  if let err = error
  {
    print(err)
  }else{
    // do your stuff with the receipt data here
    if let receipt = try? InAppReceipt.localReceipt() {
      // ...
    }
  }
}
```



## Check if user has purchased products

Here's the fun part. For **non-consumable** product, you can check if the user has purchased it using **containsPurchase(ofProductIdentifier)** function : 

```swift
if let receipt = try? InAppReceipt.localReceipt(){
    if receipt.containsPurchase(ofProductIdentifier: "your.product.identifier"){
        // user has purchased this product
        // return true
    }
}
```



For **auto-renewable subscription**, you can check if the user has any active subscriptions with **hasActiveAutoRenewablePurchases** property :

```swift
if let receipt = try? InAppReceipt.localReceipt(){
    if receipt.hasActiveAutoRenewablePurchases {
        // user has active subscription
        // return true
    }
}
```



If you would like to be more specific, you can check if user has an active subscription of a specific product ID, until a specified date like this : 

```swift
if let receipt = try? InAppReceipt.localReceipt(){
    if receipt.hasActiveAutoRenewableSubscription(ofProductIdentifier: "your.product.identifier.", forDate: Date()) {
        // user has subscription of the product, which is still active at the specified date
        // return true
    }
}
```



 ## Check receipt original app version

If your app is a paid app and you want to make it free with in-app purchase, at the same time allowing user who have purchased the paid app previously to get the in-app purchase for free, you can use the **originalAppVersion** property to check what is the app version number when the user first downloaded the app.



// iOS means build number, // macOS means version number







