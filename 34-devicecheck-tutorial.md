## Uniquely identify iOS device using DeviceCheck (Tutorial)

Say you want to let a user do something only once from a single device. For example, to redeem a one-time reward of $100 in-game currency. How can you prevent user from doing said action more than once on their device?



Here's some way you can try :

1. Store a boolean flag in UserDefaults which indicate a user has claimed it before. But UserDefaults doesn't persist through App reinstalls, user can simply remove and reinstall the app and redeem it again.
2. Similar to UserDefaults, store a boolean flag in Keychain, it might persist through App removal/reinstalls, but [Apple made no promise](https://forums.developer.apple.com/thread/36442#281900) that this behaviour will remain in future iOS versions. As in iOS 10.3 Beta, removing an app also remove its Keychain data.
3. Use [Advertising Identifier](https://developer.apple.com/documentation/adsupport/asidentifiermanager/1614151-advertisingidentifier) to identify a user and send this data to server to check if an user has redeemed it before. However user can also reset this easily in the Settings app.
4. Rely on server user login system, which user needs to register/login to an account before redeeming. User can just keep creating dummy account / email or even phone numbers to redeem it. (I have worked in a company specializing in coupon reward redeem app, you won't believe how far a user is willing to go to keep on redeeming these reward 😂)



So... what option do we have? Fortunately, Apple has introduced [DeviceCheck](https://developer.apple.com/documentation/devicecheck) in iOS 11. It is an API that allows us to access per-device, per-developer data in an iOS device. DeviceCheck allows each developer to assign two bits (2x2) of data per-device (note: not per-app, which means if you have 10 apps, they will all share the same two bits per device!).



The two bits (named bit0 and bit1) can store a boolean value (true or false) : 
![Two bits](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/bits.png)



The DeviceCheck class in iOS allows us to generate a temporary token (which Apple server will use to identify a device), then we send this token to a backend server, and then backend server will communicate with Apple's server to retrieve the bit data or update the bit data.



![app server apple communication](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/server.png)

This post will separate into app side and server side. Most of the difficult part is on the server side.



## App side

Note that DeviceCheck will only works on physical device, **it will not work on Simulator**. You can use `.isSupported` to check if the current device support DeviceCheck API. 



You will also need to have a valid Apple Developer account, and sign your project with it.

![signing](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/signing.png)





To use DeviceCheck in your iOS project's code, simply import the DeviceCheck module, and use its token generation function.


```swift
import UIKit
import DeviceCheck

@IBAction func claimRewardTapped(_ sender: UIButton) {
  let device = DCDevice.current
  
  // check if the device supports the DeviceCheck API
  guard device.isSupported else {
    let alert = UIAlertController(title: "Unsupported device", message: "Please try in a real device instead of simulator", preferredStyle: .alert)
    let okAction = UIAlertAction(title: "OK", style: .default, handler: nil)
    alert.addAction(okAction)
    
    self.present(alert, animated: true, completion: nil)
    return
  }
  
  // generate token for DeviceCheck
  device.generateToken(completionHandler: { data, error in
    guard error == nil,
    let data = data else {
      let alert = UIAlertController(title: "Unable to generate token", message: "Please sign the app using a valid Apple Developer Account", preferredStyle: .alert)
      let okAction = UIAlertAction(title: "OK", style: .default, handler: nil)
      alert.addAction(okAction)
      
      self.present(alert, animated: true, completion: nil)
      return
    }
    
    // send the base64 encoded device token string to your server
    let token = data.base64EncodedString()
    
    // use URLSession or something to send the token string to your server API
  })
}
```

<br>



In the code, we initialize a DeviceCheck device by calling **DCDevice.current**, then we generate a token using  **.generateToken**, and then encode the raw token data using base64 encoding, and send it to backend server.



That's all to it for the iOS code! Pretty straightforward right?



## Server side

For this part, I will be using [Sinatra](http://sinatrarb.com) (a lightweight ruby web framework) for the server side code, you can use any server side language/framework  as the same concept still apply.



Before proceeding to the server code, we will first need to generate a DeviceCheck Key on Apple developer portal. Go to [https://developer.apple.com/account/ios/authkey/create](https://developer.apple.com/account/ios/authkey/create) to create a key, and check **DeviceCheck** in the Key Services list.



![create key](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/createkey.png)

Click continue, download the key file, and also copy the Key ID, we will need this later on.

![keyID](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/keyID.png)











