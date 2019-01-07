# Check if user has purchased in-app item / product using Keychain

[Ray Wenderlich's tutorial](https://www.raywenderlich.com/122144/in-app-purchase-tutorial) on how to implement in-app purchases is great, I followed their tutorial to implement in-app purchase for my app [Rapidly](https://rapidly.pro).



One crucial step I feel they skipped was how to store information about purchased products / item locally so that the app can know whether the user has purchased a certain in-app purchase item previously without having to check with Apple. This is especially useful as user might not have internet connection when using a paid feature in your app. 



Ray Wenderlich's tutorial cautioned not to use User Default, 
> Note: User defaults may not be the best place to store information about purchased products in a real application. 

This is with good reason as data in User Defaults is stored in a .plist file in the app bundle [^1], which can be easily accessed and edited by plugging the iPhone into a Mac and running a program like [iExplorer](https://macroplant.com/iexplorer). A 14 years old teen could easily trick your app into giving free in-app purchase item by just spending 10 minutes on his computer 💸, don't ever do something like setting "isItemPurchased" boolean in User Default.



The better way to do this is using [Keychain](https://developer.apple.com/library/content/documentation/Security/Conceptual/keychainServConcepts/02concepts/concepts.html#//apple_ref/doc/uid/TP30000897-CH204-TP9) , it's similar to User Default plus it comes with security protection such as data encryption and protection against external access (Other apps cannot access your app keychain item unless signed with the same provisioning profile). Aside from better security, the data saved in Keychain still remains even after the app is uninstalled whereas User Default does not. ie. If the user purchase in-app product, uninstall the app and install again, the system will still know that the user has purchased the in-app product previously.



As the [original Keychain API](https://developer.apple.com/library/content/documentation/Security/Conceptual/keychainServConcepts/iPhoneTasks/iPhoneTasks.html#//apple_ref/doc/uid/TP30000897-CH208-SW1) provided by Apple is quite complex to use, we will be using a keychain wrapper library [KeychainAccess](https://github.com/kishikawakatsumi/KeychainAccess/) for this post.



**The following steps assume you have followed Ray Wenderlich's tutorial to implement In-app purchase**.

## Step 0 - Install KeychainAccess using Cocoapods

If you didn't know what is Cocoapods, its a dependency manager (library installer wizard) for Swift/Objective-C projects. [Cocoapods installation guide here](https://cocoapods.org/#install) 

Add a new line `pod 'KeychainAccess'` to the Podfile and run `pod install` .



## Step 1 - Inform the keychain to save purchased item when user has completed purchase

Open **IAPHelper.swift** , add `import KeychainAccess` at the top.

In the **deliverPurchaseNotificationFor(identifier:)** function, remove the User Default code and replace it with keychain.

```swift
// IAPHelper.swift

private func deliverPurchaseNotificationFor(identifier: String?) {
    guard let identifier = identifier else { return }
	
    purchasedProductIdentifiers.insert(identifier)
	

    // replace the keychain service name as you like
    let keychain = Keychain(service: "com.companyname.appname.iapService")
    
    // use the in-app product item identifier as key, and set its value to indicate user has purchased it
    do 
        try keychain.set("purchased", key: identifier)
    }
    catch let error {
	    print("setting keychain to purchased failed")
	    print(error)
    }
    
    NotificationCenter.default.post(name: NSNotification.Name(rawValue: IAPHelper.IAPHelperPurchaseNotification), object: identifier)
}
```



## Step 2 - Check keychain to determine if a user has purchased the item  /product

In **IAPHelper.swift**, add a keychain checking code for purchased item identifier in the **isProductPurchased(_ productIdentifier:)** method.

```swift
// IAPHelper.swift

public func isProductPurchased(_ productIdentifier: ProductIdentifier) -> Bool {
		
	let keychain = Keychain(service: "com.companyname.appname.iapService")
	
	// if there is value correspond to the productIdentifier key in the keychain
	if let hasPurchased = try? keychain.get(productIdentifier){
		
		// the product has been purchased previously, add it to the purchasedProductIdentifiers set
		purchasedProductIdentifiers.insert(productIdentifier)
		
	} else {
		// the product has not been purchased previously, do nothing
		print("Not purchased: \(productIdentifier)")
	}
	
	return purchasedProductIdentifiers.contains(productIdentifier)
}
```

<br>

Then, in the IAPHelper **init(productIds:)** method, remove the User Default checking code altogether, so it will look like this : 
```swift
public init(productIds: Set<ProductIdentifier>) {
	productIdentifiers = productIds
	super.init()
	SKPaymentQueue.default().add(self)
}
```

<br>

There is no need to add the Keychain checking code here as the checking code is implemented in the **isProductPurchased(_ productIdentifier:)** method.



## Step 3 - Viola!

You can use **isProductPurchased(_ productIdentifier:)** to determine whether to show user paid feature or buy button. Here's an example : 

```swift
// something like this in your view controller
if(RapidProducts.store.isProductPurchased(RapidProducts.ProFeature)){
    showPaidFeature()
}
```

<br>

Now you have a functioning in-app purchase implementation with checking!



[^1]: In `<app name>/Library/Preferences/<app bundle identifier>.plist`


<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> Learn and understand how Auto Layout works</h4>
    <span style="font-size:0.8em;"> 
    Can't wrap your head around Auto Layout? <br><br>
    In this free 6 lesson email course, you will learn :
    <ol>
        <li>How Auto Layout determines the position and size of a view 📏</li>
        <li>How to solve red lines (missing / conflicting constraint) in Interface Builder❗️</li>
        <li>How to create dynamic height label and using it for dynamic layout⚡️</li>
    </ol>
    </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/448009646/submissions" method="post" data-drip-embedded-form="448009646">
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
                <input type="submit" value="Send me Lesson 1" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips to help you become a better iOS developer.<br> No Spam. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>