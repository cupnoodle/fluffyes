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



## Check if user has purchased products or has active subscription

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



According to [Apple's documentation](https://developer.apple.com/documentation/appstorereceipts/responsebody/receipt?changes=_2),  originalAppVersion refers to the **build number of your iOS app** :



![build number](https://iosimage.s3.amazonaws.com/2019/55-paid-to-free/CFBundleVersion.png)



![build number](https://iosimage.s3.amazonaws.com/2019/55-paid-to-free/buildNumber.png)



If the original app version number is smaller than the build number that was last used when the app is still paid, grant the user access to the in-app purchase.

```swift
// cast the string into integer (build number)
guard let receipt = try? InAppReceipt.localReceipt(),
      let originalBuildNumber = Int(receipt.originalAppVersion) else {
    return
}

// the last build number that the app is still a paid app
if originalBuildNumber < 37 {
    // grant user premium feature here
    print("premium granted")
}
```

<br>



## Validate receipt

TPInAppReceipt also provides methods to verify if the receipt is legit and not forged. You can verify the receipt's :

1. Bundle Identifier and Version
2. GUID Hash
3. Signature



```swift
// Verify GUID hash 
try? receipt.verifyHash()

// Verify bundle identifier and version
try? receipt.verifyBundleIdentifierAndVersion()

/// Verify signature
try? receipt.verifySignature()
```



You can verify these three steps at once using the **verify()** method : 

```swift
// Verify all at once

do {
    try receipt.verify()
} catch IARError.validationFailed(reason: .hashValidation) 
{
    // Hash validation failed
} catch IARError.validationFailed(reason: .bundleIdentifierVefirication) 
{
    // Bundle identifier verification failed
} catch IARError.validationFailed(reason: .signatureValidation) 
{
    // Signature validation
} catch {
    // Miscellaneous error
}
```



objc.io has written an excellent article which explains these validation steps in detail here : https://www.objc.io/issues/17-security/receipt-validation/ , I highly recommend giving it a read even though you are using a library to handle it.



Here are some key points from objc.io article on receipt :

> - A receipt is created and signed by Apple through the App Store.
> - A receipt is issued for a specific version of an application and a specific device.
> - A receipt is stored **locally** on the device.
> - A receipt is issued each time an installation or an update occurs.
>   - When an application is installed, a receipt that matches the application and the device is issued.
>   - When an application is updated, a receipt that matches the new version of the application is issued.
> - A receipt is issued each time a transaction occurs:
>   - When an in-app purchase occurs, a receipt is issued so that it can be accessed to verify that purchase.
>   - When previous transactions are restored, a receipt is issued so that it can be accessed to verify those purchases.



Below I will explain each of the verification steps in simplified form.



### Bundle Identifier and Version

When a receipt is issued from Apple and stored into your app bundle, the receipt contains data about the bundle identifier and bundle version (on iOS, the bundle version refers to the CFBundleVersion, which is the build number value).



The bundle identifier value should match your app bundle identifier, and the bundle version should match the build number value of the app. This verification step is to prevent the hacker to take a valid receipt from another app and replace it in your app bundle, thus bypassing the check if you didn't implement this step.



Here's an excerpt from the [.verifyBundleIdentifierAndVersion()](https://github.com/tikhop/TPInAppReceipt/blob/a2f83c0f78aebb24306b43a210e907b566842fc8/TPInAppReceipt/Source/Validation.swift#L51) method :

```swift
// check if bundle identifier value of the receipt equal to the app ones
guard let bid = Bundle.main.bundleIdentifier, bid == bundleIdentifier else
{
    throw IARError.validationFailed(reason: .bundleIdentifierVefirication)
}

// check if bundle version value of the receipt equal to the app ones
guard let v = Bundle.main.infoDictionary?["CFBundleVersion"] as? String,
            v == appVersion else
{
    throw IARError.validationFailed(reason: .bundleVersionVefirication)
}
```



### GUID Hash

**GUID** (or UUID) is an acronym for 'Globally Unique Identifier' (or 'Universally Unique Identifier'). 

On iOS, this identifier is generated using **UIDevice.current.identifierForVendor.uuid**. 



Here is the simplified diagram on how the hash comparison works :



![hash verification](https://iosimage.s3.amazonaws.com/2020/80-validate-local-receipt/hash_verification.png)



The receipt contains data of the app's bundle identifier, an opaque value (you can think of it like a salt before hashing), and the resulting SHA1 hash value, which you can compare with later.



We first generate the Device GUID using UIDevice's **identifierForVendor.uuid** , then concatenate the GUID value with the Opaque value, then concatenate with the Bundle Identifier.



Then we take the concatenated value and generate a SHA1 hash, then we compare this generated hash with the SHA1 hash value that is present in the receipt data. This two value should be equal, if they are not equal, this means that the receipt is not valid (which might be forged by the hacker).



### Signature

A legitimate receipt is signed by Apple, using their own **private key** (which only Apple have it). 



A signature is generated when a private key is used to sign a data / file, you can think of it like a hash that is generated when you feed an input to a hashing algorithm. We can then use the **public key** (list of Apple's public key/certificate available to download here : https://www.apple.com/certificateauthority/) to verify if the signature matches by using it to read the signed data.



