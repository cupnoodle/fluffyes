# Send user to App settings using Settings URL



From the previous [Core Location tutorial](https://fluffy.es/current-location/), we need to prompt an authorization window for user to allow location access in order to get the location data : 

![location permission](https://iosimage.s3.amazonaws.com/2019/45-current-location/permissionAlert.png)





Chances are, the user might decline it on the first run. The bad news is that after the user has declined the prompt, the prompt won't show again even if we call **requestWhenInUseAuthorization()** later on. 



The only way a user can grant location access (or camera, microphone etc) to an app after declining the first prompt is to go to the Settings app and manually allow permission there.

![manually go to Settings](https://iosimage.s3.amazonaws.com/2019/47-app-settings/settingsManual.gif)



This would require user to open Settings app, scroll down the long list to your app and tap multiple times to allow permission, which can cause annoyance to user. To reduce the cognitive load on the user, we can show an alert with button that links to the app permission section directly using **SettingsURL** when the app detect that the permission has not been granted.



// show code that shows alert and move user to settings url

