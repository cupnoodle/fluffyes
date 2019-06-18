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







