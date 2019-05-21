# Remote push notification in Testflight / App Store

> I have tried sending push notification to the app running on iPhone and it was sent successfully. But when I upload the app to Testflight for testing, no push notification is shown! 



Ever encounter the situation above? The reason why the push notification doesn't get sent to Testflight / App Store is that **for Testflight / App Store, the production APNS certificate is used** as opposed to the development APNS certificate. And for different environment (dev/ production), the device token will be different even on the same exact app on the same device, so you might need to send the device token from the app to your server (or push notification service provider) again.



Here's the flow of how push notification is being sent : 

![remote push notification flow](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/pushflow.png)



If you have encountered the issue mentioned above, most likely you are using a development certificate to send push notification. To send push notification to Testflight / App Store, we will need to use a production push certificate and send the push notification payload to production Apple Push Notification Server (APNS, https://api.push.apple.com) .





## Getting the production push certificate

Similar to generating development push certificate, head over to https://developer.apple.com/account , then click **'Certificates, IDs & Profiles**' on the left side bar. Then click '**App IDs**' , and select your App name, then click '**Edit**' .



![app ID](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/appID.png)

![edit app ID](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/editAppID.png)



Then create the production certificate for push notification : 

![production push certification create](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/createCertificate.png)



Then follow the instruction from Apple to create a Certificate Signing Request (CSR) file, and generate the production push notification certificate. 

