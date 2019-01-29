# Get current location using Core Location (Tutorial)



Last month, I added a show nearby train station feature to my train app [Rapidly](https://rapidkl.app) . I've stumbled across few issues while following some tutorials online on how to implement Core Location, in this post we will look into how to retrieve current location of user using Core Location and some common pitfalls of it.



We will use CLLocationManager (the 'CL' stands for Core Location) to handle location stuff. Here's the initialization code we'll be using : 

```swift
// ViewController.swift

import UIKit
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
// when user tap on a button to get location
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



Both of these methods will instruct the iOS device to retrieve its current location data, and once the location data is retrieved, the delegate method **didUpdateLocations locations:** will be called. The difference is that .requestLocation() will only call the `didUpdateLocations locations:` once , whereas .startUpdatingLocation() will keep on calling `didUpdateLocations locations:` every few seconds or whenever there's location change until you stop it by calling locationManager.stopUpdatingLocation.



### .requestLocation() flow

![request Location](https://iosimage.s3.amazonaws.com/2019/45-current-location/requestLocation.png)

`didUpdateLocations locations:` is called only once when we use .requestLocation.



.requestLocation() will only pass one location to the didUpdateLocations **locations** array. 

```swift
extension ViewController : CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        // .requestLocation will only pass one location to the locations array
        // hence we can access it by taking the first element of the array
        if let location = locations.first {
            self.latitudeLabel.text = "\(location.coordinate.latitude)"
            self.longitudeLabel.text = "\(location.coordinate.longitude)"
        }
    }
}
```

<br>



### .startUpdatingLocation() flow

![start updating location flow](https://iosimage.s3.amazonaws.com/2019/45-current-location/updateLocation.png)

`didUpdateLocations locations:` is called whenever there is update to location / every few seconds until we call locationManager.stopUpdatingLocation.



.startUpdatingLocation() might pass more than one location to the didUpdateLocations **locations** array. 


```swift
extension ViewController : CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        // .startUpdatingLocation might pass more than one location to the locations array
      for location in locations {
        // do stuff with each location data
      }
    }
}
```

<br>



## Handling Error

There might be a possibility that the app failed to retrieve location data (after calling .requestLocation or .startUpdatingLocation), when this happen, the **didFailWithError error:** will be called.

```swift
func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
    // might be that user didn't enable location service on the device
    // or there might be no GPS signal inside a building
  
    // might be a good idea to show an alert to user to ask them to walk to a place with GPS signal
}
```

<br>



## 'didUpdateLocation' didn't get called even after I have set delegate! What happened?

I saw this question asked a lot in Apple developers forum, and noticed most of them requested location data right after requesting permission like this :

```swift
locationManager.requestWhenInUseAuthorization()
locationManager.requestLocation()
```

<br>



The problem with this is that **.requestWhenInUseAuthorization()** is asynchronous, meaning after the permission alert is shown, the next line of code will be executed **before** user has tap on the alert.



Don't do this :

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    locationManager.delegate = self
  
		locationManager.requestWhenInUseAuthorization()
    // after showing the permission dialog, the program will continue executing the next line before the user has tap 'Allow' or 'Disallow'
    
    // when the requestLocation() function is called, the status is still .undetermined, hence iOS won't start requesting location
    locationManager.requestLocation()
}

```

<br>



Instead, do it like this : 

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    locationManager.delegate = self
  
		locationManager.requestWhenInUseAuthorization()
    // after showing the permission dialog, the program will continue executing the next line before the user has tap 'Allow' or 'Disallow'
    
    // if previously user has allowed the location permission, then request location
    if(CLLocationManager.authorizationStatus() == .authorizedWhenInUse || CLLocationManager.authorizationStatus() == .authorizedAlways){
        locationManager.requestLocation()
    }
}

// After user tap on 'Allow' or 'Disallow' on the dialog
func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
  if(status == .authorizedWhenInUse || status == .authorizedAlways){
    manager.requestLocation()
  }
}
```

<br>



## Remember to test CoreLocation function on real device

If you test CoreLocation functions in simulator, simulator will use simulated location data and return it almost instantly, as your Mac doesn't have GPS functionality.

![simulator gif](https://iosimage.s3.amazonaws.com/2019/45-current-location/simulatorInstant.gif)



You can change the location coordinate for the simulator using **Debug** > **Custom Location** .

![location simulator](https://iosimage.s3.amazonaws.com/2019/45-current-location/locationData.png)



If you test it in real device, it might take up to 10 seconds, depending on the accuracy you have set for the CLLocationManager.



## requestLocation() might take longer to retrieve location data

As mentioned in this [WWDC video about CoreLocation](https://developer.apple.com/videos/play/wwdc2017/713/) (at around 11:55), **requestLocation()** is a convenient method provided by Apple which under the hood will run **startUpdatingLocation()** , retrieve multiple location data, and select the most accurate one to pass to delegate, and call **stopUpdatingLocation()**. This process can take up to 10 seconds (which is around the timeout limit) if it can't decide which location data is the best.



If speed of retrieval is crucial for your app and you are willing to handle the start/stopping of location data retrieval and filtering of location data, use **startUpdatingLocation()** instead.



## Accuracy 

You can specify how accurate you want the location data to be by specifying the **desiredAccuracy** property for the CLLocationManager before requesting location data.



```swift
// location data accurate until hundred meters
locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
```

<br>

Here's a list of accuracy type : 

![Accuracy type](https://iosimage.s3.amazonaws.com/2019/45-current-location/accuracyType.png)



From [Apple's documentation on desiredAccuracy](For iOS and macOS, the default value of this property is `kCLLocationAccuracyBest`. For watchOS, the default value is `kCLLocationAccuracyHundredMeters`.), if you didn't set the desiredAccuracy, the default values are as follow : 

> For iOS and macOS, the default value of this property is `kCLLocationAccuracyBest`. For watchOS, the default value is `kCLLocationAccuracyHundredMeters`.



One thing to note is that higher accuracy will require longer time to retrieve location data. Using **kCLLocationAccuracyHundredMeters** to call requestLocation() took 1 second on my phone, whereas **kCLLocationAccuracyBest** took 10 seconds.



I suggest to use the least accurate accuracy your app need (hundred meters is usually good enough unless your app is like Waze / Google Maps / Emergency app), this can save time on retrieving location.



## Retrieving location data when the app is in the background

To enable your app to retrieve location data while in background, you'll need to :

1. Select your project > Capabilities > turn on Background modes and check 'Location updates'

![background mode](https://iosimage.s3.amazonaws.com/2019/45-current-location/backgroundModes.png)



2. Request always authorization for the CoreLocation manager

   ```swift
   locationManager.requestAlwaysAuthorization()
   ```

   <br>

   The user would need to tap 'Always allow' when prompted permission , and the authorization status need to be **.authorizedAlways**

   

3. Start update location and that's it!

   ```swift
   locationManager.startUpdatingLocation()
   ```







