# Remote push notification in Testflight / App Store

> I have tried sending push notification to the app running on iPhone and it was sent successfully. But when I upload the app to Testflight for testing, no push notification is shown! 



Ever encounter the situation above? The reason why the push notification doesn't get sent to Testflight / App Store is that for Testflight / App Store, the production APNS certificate is used as opposed to the development APNS certificate. And for different environment (dev/ production), the device token will be different even on the same exact app on the same device, so you might need to send the device token from the app to your server (or push notification service provider) again.






