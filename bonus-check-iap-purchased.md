# How to check if a user has purchased in-app purchase

(The following paragraphs only applies to non-consumable product)



After user has purchased in-app purchase, how can the app know that the user has purchased the in-app purchase when the user return to the app later on? The app needs to know that if the user has purchased in before, and then show or enable the premium content / feature when the user reopen the app again.



One straightforward approach is to use UserDefaults to save the data of user purchase, maybe use a key "hasPurchasedPro" on UserDefaults, and set its boolean value to true after user has completed the purchase : 



```swift
let hasPurchasedPro = UserDefaults.standard.bool(forKey: "hasPurchasedPro")

if (hasPurchasedPro) {
  // enable premium feature here
}
```

<br>



However, this approach might make your app more vulnerable to piracy, as user can easily modify the UserDefaults data **without** jailbreaking their phone! 



[Data in User Defaults are stored in a .plist file](https://fluffy.es/persist-data/#userdefaults) in the app bundle, which can be easily accessed and edited by plugging the iPhone into a Mac and running a program like [iExplorer](https://macroplant.com/iexplorer). A 14 years old teen could easily trick your app into giving free in-app purchase item by just spending 10 minutes on his computer 💸, don't ever do something like setting "isItemPurchased" boolean in User Default! 



A more secure way is to use **Keychain** to store this kind of info, Keychain is similar to UserDefaults, with key/values option, plus it comes with security protection such as data encryption and protection against external access. I have written a detailed guide on how to use Keychain to check if user has purchased in-app purchase before here : https://fluffy.es/check-purchased-iap-using-keychain/



Unarguably, the best way to check if a user has purchased an in-app purchase item is still using **receipt**. After a user download an app, purchase an in-app purchase or restore an earlier purchase, the app will download a receipt file from App store in ASN1 + base64 encoded format, this receipt file is stored in this path : **Bundle.main.appStoreReceiptURL?.path** . The receipt contains the purchase histories of all non-consumable product, non-renewing subscription and auto-renewing subscription, including their purchase date.



**Receipt should be the ultimate source of truth on whether a user has made a purchase** , but parsing the receipt data is moderately difficult as you need to know how to decode the old ASN1 formatted data, and you might need to do some cryptographic verification to ensure that the receipt is legit (some jailbroken user can put a fake receipt into your app to attempt to trick your app into thinking they have bought a pro feature).



Fortunately, there's many open source library in Github that help us handle the receipt parsing and validation. My favorite receipt parsing / validating library is this : https://github.com/tikhop/TPInAppReceipt , I have used it in all of my iOS apps, and [have contributed to it for the signature validation](https://github.com/tikhop/TPInAppReceipt/pull/37) part of the receipt. 



The takeaway of this email is that : 

1. **Don't use UserDefaults** to check if user has purchased product
2. If possible, **always opt to check receipt data** directly for checking purchase histories.

