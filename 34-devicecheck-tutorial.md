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

For this part, I will be using Ruby language for the server side code, you can use any server side language/framework as the same concept still apply.



Before proceeding to the server code, we will first need to generate a DeviceCheck Key on Apple developer portal. Go to [https://developer.apple.com/account/ios/authkey/create](https://developer.apple.com/account/ios/authkey/create) to create a key, and check **DeviceCheck** in the Key Services list.



![create key](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/createkey.png)

Click continue, download the key file, and also copy the Key ID, we will need this later on.

![keyID](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/keyID.png)

You can open the key file (.p8) using TextEdit App to get its content.



### Authorization Token (JWT Token)

This is the most confusing part of communicating with Apple's DeviceCheck API. To verify that the server is owned by you, you will need to use the key file downloaded earlier to generate a [JSON web token](https://jwt.io) (JWT). JWT  is an encoded string generated with header and payload.

Here is how JWT is generated : 
![JWT generation](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/jwt.png)



Header and Payload is a JSON type. 



In the header, we will include **alg** key  (algorithm) with value of **ES256**, Apple mentioned in documentation that ES256 algorithm is used so we will use this. The **kid** key means key_id, this is the Key ID you copied in previous part.



In the payload, we will include **iss** key (issuer) with value of your Apple developer team's ID, you can get this value here : [https://developer.apple.com/account/#/membership](https://developer.apple.com/account/#/membership) . The **iat** key means issued_at, we will use the Unix Timestamp of current time (**in seconds**).



The ruby code to generate jwt token: 

```ruby
require 'openssl'
require 'http'
require 'jwt'
require 'securerandom'

def jwt_token
  # use the content of the .p8 key file you downloaded, it should look like this :
	#-----BEGIN PRIVATE KEY-----
	#ILIKEFOXES
  #-----END PRIVATE KEY----
  private_key = ENV['DEVICE_CHECK_KEY_STRING']
  
  # the Key ID you saw earlier
  key_id = ENV['DEVICE_CHECK_KEY_ID']
  
  # Team ID of your Apple developer account
  team_id = ENV['DEVICE_CHECK_TEAM_ID']

  # Elliptic curve key, used to encrypt the JWT
  ec_key = OpenSSL::PKey::EC.new(private_key)
  jwt_token = JWT.encode({iss: team_id, iat: Time.now.to_i}, ec_key, 'ES256', {kid: key_id,})
end
```

<br>



We then use this JWT in the HTTP Header (Authorization field) :

```
"Authorization" : "Bearer eyJhbGciOiJIU...."
```

<br>



We need to include this HTTP header when communicating with Apple's DeviceCheck API.



### Query Two Bits

To get the two bits state of a device, we will make a HTTP request to https://api.devicecheck.apple.com/v1/query_two_bits (this is the production server for app live in App Store or Testflight). If you are in development mode (plugging iPhone to Mac), use this API instead : https://api.development.devicecheck.apple.com/v1/query_two_bits .

![query](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/query.png)



We will make a POST request to Apple's server with JSON containing device_token, timestamp and transaction_id.

**device_token** is the 64 base encoded token generated from the app.

**timestamp** is the current time in Unix Timestamp when you send the request to Apple, **in milliseconds** .

**transaction_id** is a unique string, this can be any string you want, as long as each HTTP request you make to Apple's server uses a different transaction_id.



The ruby code to perform the bits query: 

```ruby
require 'openssl'
require 'http'
require 'jwt'
require 'securerandom'

def query_two_bits(device_token)
  payload = {
    'device_token' => device_token,
    'timestamp' => (Time.now.to_f * 1000).to_i,
    'transaction_id' => SecureRandom.uuid
  }

  query_url = 'https://api.development.devicecheck.apple.com/v1/query_two_bits'
  response = HTTP.auth("Bearer #{jwt_token}").post(query_url, json: payload)

  # if there is no bit state set before, 
  # Apple will return the string 'Bit State Not Found' instead of json

  # if the bit state was set before, json below will be returned
  #{"bit0":false,"bit1":false,"last_update_time":"2018-10"}
end
```

<br>



If the bit state of the device haven't been set before, Apple server will return HTTP response with status 200 and a text of 'Bit State Not Found'.



If the bit state of the device has been set before, Apple server will return HTTP response with status 200 containing JSON.

![Query JSON](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/query2.png)



### Update Two Bits

To update the two bits state of a device, we will make a HTTP request to https://api.devicecheck.apple.com/v1/update_two_bits (this is the production server for app live in App Store or Testflight). If you are in development mode (plugging iPhone to Mac), use this API instead : https://api.development.devicecheck.apple.com/v1/update_two_bits .



![Update bits](https://iosimage.s3.amazonaws.com/2018/34-devicecheck-tutorial/update.png)



We will make a POST request to Apple's server with JSON containing device_token, timestamp, transaction_id, bit0 and bit1.



**device_token** is the 64 base encoded token generated from the app.

**timestamp** is the current time in Unix Timestamp when you send the request to Apple, **in milliseconds** .

**transaction_id** is a unique string, this can be any string you want, as long as each HTTP request you make to Apple's server uses a different transaction_id.

**bit0** is a boolean, you can set this to either true or false.

**bit1** is a boolean, you can set this to either true or false.



The ruby code to perform the bits update :

```ruby
require 'openssl'
require 'http'
require 'jwt'
require 'securerandom'

def update_two_bits(device_token, bit_zero, bit_one)
  payload = {
    'device_token' => device_token,
    'timestamp' => (Time.now.to_f * 1000).to_i,
    'transaction_id' => SecureRandom.uuid,
    'bit0': bit_zero, # true / false
    'bit1': bit_one # true / false
  }

  response = HTTP.auth("Bearer #{jwt_token}").post(settings.update_url, json: payload)
  # Apple will return status 200 with blank response body if the update is successful
end
```

<br>



Apple's server will return a blank HTTP response with status 200 if the update is successful.





## Applying DeviceCheck on redeeming reward

In your server code, you can make an API endpoint say '/redeem' , the iOS app will make a HTTP request (containing device token) to this endpoint (https://example.com/redeem) , then the server will use the device token and check with Apple's server. If bit0 is not set or is false, user can redeem the reward, else user can't. Server can set back a JSON telling the app that user can/can't claim the reward. After user claim the reward, server will make another request to Apple's server to update bits (set bit0 to true).



DeviceCheck is very useful and is the only legit way to uniquely identify an iOS device (as of iOS 12.1). However there is only two bits to use and **this is shared among all the apps** ,not two bits per app. If you have 10 apps, all of them will get the same bit data.









