## DeviceCheck Tutorial

Say you want to let a user do something only once from a single device. For example, to redeem a one-time reward of $100 in-game currency. How can you prevent user from doing said action more than once on their device?



Here's some way you can try :

1. Store a boolean flag in UserDefaults which indicate a user has claimed it before. But UserDefaults doesn't persist through App reinstalls, user can simply remove and reinstall the app and redeem it again.
2. Similar to UserDefaults, store a boolean flag in Keychain, it might persist through App removal/reinstalls, but [Apple made no promise](https://forums.developer.apple.com/thread/36442#281900) that this behaviour will remain in future iOS versions. As in iOS 10.3 Beta, removing an app also remove its Keychain data.
3. Use [Advertising Identifier](https://developer.apple.com/documentation/adsupport/asidentifiermanager/1614151-advertisingidentifier) to identify a user and send this data to server to check if an user has redeemed it before. However user can also reset this easily in the Settings app.
4.  Rely on server user login system, which user needs to register/login to an account before redeeming. User can just keep creating dummy account / email or even phone numbers to redeem it. (I have worked in a company specializing in coupon reward redeem app, you won't believe how persistent a user can be on redeeming these 😂)



So... what option do we have? Fortunately, Apple has introduced [DeviceCheck](https://developer.apple.com/documentation/devicecheck) in iOS 11. It is an API that allows us to access per-device, per-developer data in an iOS device. DeviceCheck allows each developer to assign two bits (2x2) of data per-device (note: not per-app, which means if you have 10 apps, they will all share the same two bits per device!).



The DeviceCheck class in iOS allows us to generate a temporary token (which Apple server will use to identify a device), and then we send this token to a backend server, and the backend server will retrieve the bit data or update the bit data by communicating with Apple server.



[Insert graph here]





