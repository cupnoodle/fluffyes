# Remote push notification in Testflight / App Store

> I have tried sending push notification to the app running on iPhone and it was sent successfully. But when I upload the app to Testflight for testing, no push notification is shown! 



Ever encounter the situation above? The reason why the push notification doesn't get sent to Testflight / App Store is that **for Testflight / App Store, the production APNS certificate is used** as opposed to the development APNS certificate. And for different environment (dev/ production), the device token will be different even on the same exact app on the same device, so you might need to send the device token from the app to your server (or push notification service provider) again.



Here's the flow of how push notification is being sent : 

![remote push notification flow](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/pushflow.png)



If you have encountered the issue mentioned above, most likely you are using a development certificate to send push notification. To send push notification to Testflight / App Store, we will need to use a production push certificate and send the push notification payload to production Apple Push Notification Server (APNS, https://api.push.apple.com) .



<div class="post-subscribe">
  <script async data-uid="82f4a0256c" src="https://f.convertkit.com/82f4a0256c/b60b8439b5.js"></script>
</div>


## Getting the production push certificate

Similar to generating development push certificate, head over to https://developer.apple.com/account , then click **'Certificates, IDs & Profiles**' on the left side bar. Then click '**App IDs**' , and select your App name, then click '**Edit**' .



![app ID](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/appID.png)

![edit app ID](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/editAppID.png)



Then create the production certificate for push notification : 

![production push certification create](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/createCertificate.png)



Then follow the instruction from Apple to create a Certificate Signing Request (CSR) file, and generate the production push notification certificate. Click **Download**, and **double click the downloaded .cer file** to save it to your local Keychain.

 

![download](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/downloadProd.png)



## Generating .p12 certificate file for push provider

Open Keychain Access, select '**My Certificates**' on the left bottom side bar

![my certificates](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/myCerts.png)

Find the production push certificate you have downloaded earlier (Apple Push Services, without 'development'). Expand the certificate, then select both the certificate and key (hold 'shift' key to select multiple). Then right click and select "Export 2 items"



![export 2 items](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/export2.png)



Make sure ".p12" is selected, then click 'Save'.

![save as p12](https://iosimage.s3.amazonaws.com/2019/54-remote-push-testflight/saveP12.png)



There will be a password dialog, if you don't want to set a password for the .p12 certificate, leave it blank and press 'OK'.



## Converting .p12 into .pem file

Some push provider might require a .pem certificate file to send push notification, we can convert a .p12 file into .pem file using OpenSSL in terminal.



In terminal, navigate to the location of the .p12 certificate, then type in the following command to convert : 

```bash
openssl pkcs12 -nodes -clcerts -in yourcert.p12 -out <filename>.pem
```

<br>



If you want to add a passphrase to the output pem file for extra security : 

```bash
openssl pkcs12 -clcerts -in yourcert.p12 -out <filename>.pem
```

<br>





With the production push certificate, now you should be able to send push notification to app in Testflight or App Store. Note that the device tokens will be different than development ones, and remember to send the push payload to the production Apple Push Notification Server (https://api.push.apple.com), instead of the development ones.



<div class="post-subscribe">
  <script async data-uid="82f4a0256c" src="https://f.convertkit.com/82f4a0256c/b60b8439b5.js"></script>
</div>



