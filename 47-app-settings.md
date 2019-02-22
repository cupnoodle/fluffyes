# Open settings app using openSettingsURLString

<s>TL;DR</s>  [Executive summary](https://twitter.com/patio11/status/1087154089467600897) : 

```swift
// open the app permission in Settings app
UIApplication.shared.open(URL(string: UIApplication.openSettingsURLString)!, options: [:], completionHandler: nil)
```





From the previous [Core Location tutorial](https://fluffy.es/current-location/), we need to prompt an authorization window for user to allow location access in order to get the location data : 

![location permission](https://iosimage.s3.amazonaws.com/2019/45-current-location/permissionAlert.png)





Chances are, the user might decline it on the first run. The bad news is that after the user has declined the prompt, the prompt won't show again even if we call **requestWhenInUseAuthorization()** later on. 



The only way a user can grant location access (or camera, microphone, push notification etc) to an app after declining the first prompt is to go to the Settings app and manually allow permission there.

![manually go to Settings](https://iosimage.s3.amazonaws.com/2019/47-app-settings/settingsManual.gif)



This would require user to open Settings app, scroll down the long list to your app and tap multiple times to allow permission, which can cause annoyance to user. To reduce the cognitive load on the user, we can show an alert with button that links to the app permission section directly using **URL(string: UIApplication.openSettingsURLString)!** .



```swift
// called when the authorization status is changed for the core location permission
func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
    print("location manager authorization status changed")

    if(status == .denied){
        let alert = UIAlertController(title: "Location access required to get your current location", message: "Please allow location access", preferredStyle: .alert)
        let settingsAction = UIAlertAction(title: "Settings", style: .default, handler: {action in

            // open the app permission in Settings app
            UIApplication.shared.open(URL(string: UIApplication.openSettingsURLString)!, options: [:], completionHandler: nil)
        })

        let cancelAction = UIAlertAction(title: "Cancel", style: .default, handler: nil)

        alert.addAction(settingsAction)
        alert.addAction(cancelAction)
      
        alert.preferredAction = settingsAction

        self.present(alert, animated: true, completion: nil)
    }
}
```

<br>



This would result the following output : 


![Settings Jump](https://iosimage.s3.amazonaws.com/2019/47-app-settings/settingsJump.gif)



<br>

<script async data-uid="5b0c2b4f98" src="https://f.convertkit.com/5b0c2b4f98/8324d183a4.js"></script>

