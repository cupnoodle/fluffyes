# Get current location using Core Location (Tutorial)



Last month, I added a show nearby train station feature to my train app [Rapidly](https://rapidkl.app) . I've stumbled across few issues while following some tutorials online on how to implement Core Location, in this post we will look into how to retrieve current location of user using Core Location and some common pitfalls of it.



We will use CLLocationManager (the 'CL' stands for Core Location) to handle location stuff. Here's the initialization code we'll be using : 

```swift
import CoreLocation

class ViewController: UIViewController {
    
    let locationManager = CLLocationManager()
  
     override func viewDidLoad() {
        super.viewDidLoad()
        locationManager.delegate = self
    }
}

extension ViewController: CLLocationManagerDelegate {
  // handle delegate methods of location manager here
}
```

<br>

Here's the overall flow of retrieving user location : 

![Location flow](https://iosimage.s3.amazonaws.com/2019/45-current-location/locationFlow.png)



## Requesting location permission from user

Before requesting location data of the user, we will need to request permission to use location service from the user.



We can request location permission by calling **.requestWhenInUseAuthorization()**  to get location data when the app is active, or **.requestAlwaysAuthorization()** to get location data when the app is active or in background (this is how Waze / Google Maps can continue get your location data even when they are in background).



```swift
// request permission to get user location when the app is in use (ie. active)
// this will show an alert box to the user
locationManager.requestWhenInUseAuthorization()

// request permission to get user location when the app is in use or in background
// this will show an alert box to the user
locationManager.requestAlwaysAuthorization()
```

<br>



One thing to note before requesting permission is that we need to specify the reason why  we are requesting to use location data of the user. We can specify the reason inside the **Info.plist**.



![location usage](https://iosimage.s3.amazonaws.com/2019/45-current-location/usagePlist.png)



In the **Info.plist**, hover any row and click the "+" button. Then type in '"Privacy - Location" and use the autocomplete to select the correct key from the list. 

**Privacy - Location When In Use Usage Description** is for .**requestWhenInUseAuthorization()**

**Privacy - Location Always and When In Use Usage Description** is for **.requestAlwaysAuthorization()**



If your app need to support iOS 10 and earlier and require always authorization, you will need to add the key **Privacy - Location Always Usage Description** together with the 'Privacy - Location Always and When In Use Usage Description ' .



After selecting the correct key, type in the reason into the 'Value' column :

![plist value](https://iosimage.s3.amazonaws.com/2019/45-current-location/plistValue.png)



If you are not sure if the key you have typed in is correct, you can view the source code of Info.plist and input the Key manually.



Right click 'Info.plist' and select 'Open As > Source Code'.

![open as source code](https://iosimage.s3.amazonaws.com/2019/45-current-location/openAsSourceCode.png)



Then type in the Key and Value using `<key></key>` and `<string></string>`

![plist source code](https://iosimage.s3.amazonaws.com/2019/45-current-location/plistSourceCode.png)



The keys are **NSLocationWhenInUseUsageDescription**, **NSLocationAlwaysAndWhenInUseUsageDescription** and **NSLocationAlwaysUsageDescription** (for iOS 10 and earlier).



The text you inputed to the plist will be shown in the request permission alert :

![permission alert](https://iosimage.s3.amazonaws.com/2019/45-current-location/permissionAlert.png)

This alert will be shown after .requestWhenInUseAuthorization() or .requestAlwaysAuthorization() is called for the first time (when user haven't tap 'Allow' or 'Dont Allow').



After user tap 'Allow' or 'Disallow', the delegate method **didChangeAuthorization status:** will be called.

```swift
extension ViewController : CLLocationManagerDelegate {
    // called when the authorization status is changed for the core location permission
    func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
        print("location manager authorization status changed")
    }
}
```

<br>



One thing to note is that this delegate method (didChangeAuthorization status) will always be called after the line `locationManager.delegate = self`, even when the permission dialog haven't been shown to user before.



Here's the possible case for the authorization status : 

```swift
extension ViewController : CLLocationManagerDelegate {
    // called when the authorization status is changed for the core location permission
    func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
        print("location manager authorization status changed")
        
        switch status {
        case .authorizedAlways:
            print("user allow app to get location data when app is active or in background")
        case .authorizedWhenInUse:
            print("user allow app to get location data only when app is active")
        case .denied:
            print("user tap 'disallow' on the permission dialog, cant get location data")
        case .restricted:
            print("parental control setting disallow location data")
        case .notDetermined:
            print("the location permission dialog haven't shown before, user haven't tap allow/disallow")
        }
    }
}
```

<br>



We can check the current authorization status using **CLLocationManager.authorizationStatus()** :

```swift
@IBAction func getCurrentLocationTapped(_ sender: Any) {
    retriveCurrentLocation()
}

func retriveCurrentLocation(){
    let status = CLLocationManager.authorizationStatus()

    if(status == .denied || status == .restricted || !CLLocationManager.locationServicesEnabled()){
        // show alert to user telling them they need to allow location data to use some feature of your app
        return
    }

    // if haven't show location permission dialog before, show it to user
    if(status == .notDetermined){
        locationManager.requestWhenInUseAuthorization()

        // if you want the app to retrieve location data even in background, use requestAlwaysAuthorization
        // locationManager.requestAlwaysAuthorization()
        return
    }
    
    // at this point the authorization status is authorized
    // request location data once
    locationManager.requestLocation()
  
    // start monitoring location data and get notified whenever there is change in location data / every few seconds, until stopUpdatingLocation() is called
    // locationManager.startUpdatingLocation()
}
```

<br>



We will also have to check if the location services is enabled on the user phone (using **CLLocationManager.locationServicesEnabled()** ), user might have turned off location services capability on the device (globally, which affect all apps) but allow your app to access location data. In this case, the delegate method 'locationManager:didFailWithError:' will be called when the app tries to retrieve location data,  with the `kCLErrorDenied` error code.



## Retrieving user current location

After ensuring the location services is enabled and the authorization status is authorized, we can start to request location by calling **locationManager.requestLocation()** (one-time delivery of location data), or have an ongoing stream of location updates using **locationManager.startUpdatingLocation()**. 



Both of these methods will instruct the iOS device to retrieve its current location data, and once the location data is retrieved, the delegate method **didUpdateLocations locations:** will be called. The difference is that .requestLocation() will only call the `didUpdateLocations locations:` once , whereas .startUpdatingLocation() will keep on calling `didUpdateLocations locations:` every few seconds / whenever there's location change of the device until you stop it by calling locationManager.stopUpdatingLocation.


























// Remember asynchronous, it takes quite some time for the phone to retrieve the location data, hence it will continue executing code below first



// requestLocationDataOnce takes more time than the start / stop requesting location data delegate



// higher accuracy location data requires longer time to retrieve, use the least accuracy you need to save time



