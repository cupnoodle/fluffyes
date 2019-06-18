# Solving Keychain Access can't export to .p12

You have successfully retrieved a .cer certificate file for push notification, and the next step is to generate a .p12 certificate so you can use it on OneSignal, Urban Airship, Firebase FCM etc, but when you open the .cer file Keychain Access and select 'Export', there's no .p12 option! (The .p12 option is grayed out)



![p12 grayed out](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/p12_grayed.png)



One of the quick fix for this is to select the '**My Certificates**' tab on the left side pane in Keychain Access :

![my certificates](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/my_certs.png)



And then expand the certificate (usually starts with the name '**Apple Push Services**' or '**Apple Development Push Services**'),  by clicking the arrow beside the certificate icon. 



Select both the certificate and key, then right click and select 'Export 2 items', you should see the .p12 option is being selected 🙌.

![select 2 items](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/select2.png)



![export 2 items](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/export2.png)



## Wait, my push certificate doesn't appear when I select 'My Certificates' ?

If your push certificate doesn't appear when you select 'My Certificates' , most probably that the Mac you are using don't have the corresponding private key for the push certificate.



When we communicate with Apple Push Notification server (APNS), either from our own server or push providers such as OneSignal, Firebase FCM, Urban Airship etc, we need to send in both the certificate and key to the APNS. (except if you are using .p8 key, I will write another article for this 😆)



The .p12 file contains both the certificate and key :

![p12 file](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/p12.png)



If your push certificate doesn't appear in 'My Certificates', you would need to go through the Certificate Signing Request (CSR) again, to regenerate the private key, and generate a new set of certificate that correspond to the new private key.



We can head over to [https://developer.apple.com/account](Apple developer account) page to generate a new push certificate again. After login, click the "App IDs and Certificate" link.



![go to cert](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/goToCertIden.png)



Then choose 'Identifiers' and click your app ID (if it doesn't exist yet, you should create one that match your app bundle ID).

![identifier app](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/identifierApp.png)



Inside the app ID, scroll down until you find 'Push Notifications', then click '**Edit**'.

![edit push cert](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/editPushCert.png)



Select the type of push certificate you want to create, **Development** is used for Xcode build of your app, whereas **Production** is for Testflight / App Store build.



![create push certificate](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/createPushCert.png)



After clicking 'Create Certificate' , Apple will ask for a **certificate signing request** (CSR). During generation of CSR, your Mac will create a private key, and Apple will generate a push certificate .cer file corresponding to this private key.



In your Mac, open **Keychain Access**, and on the top menu, select **Keychain Access** > **Certificate Assistant** > **Request a Certificate From a Certificate Authority** . This will launch the certificate assistant app.



In the certificate assistant app, fill in  **Common Name** , this will be the name of your private key in Keychain Access later, and select '**Saved to Disk**' , so that it will generate a **.certSigningRequest** file.



We then upload this file to Apple in the Push Certificate creation page.

![upload CSR to Apple](https://iosimage.s3.amazonaws.com/2019/57-keychain-export-p12/createCSR.png)



After uploading the CSR file, you will obtain the .cer file, double click this file and you should be able to view it in **My Certificates** tab in Keychain Access, and export it to .p12 🙌.



























