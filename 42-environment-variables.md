# Using launch arguments / environment variables to simulate in-app purchase purchased and etc



If you have experience programming in other language like Ruby , Python, Java etc, you might have used environment variable to store secrets. There's environment variable in iOS as well, these are injected to your app in runtime.



Here's how I learned to use launch argument / environment variable the hard way :



In my app [Komuter](https://komuter.app), users will need to purchase in-app purchase to view upcoming trains arrival time.

![purchase](https://iosimage.s3.amazonaws.com/2018/42-environment-variables/purchase.png)



In this screen, I call the function `hasPurchasedPro()` to check if the user has purchased in-app purchase, if user has bought in-app purchase, the buy button and blurred text will be hidden, and the actual upcoming train time will be shown.



```swift
func hasPurchasedPro(){
  // check Keychain to see if the purchase data exist or not
  // return true if it exist, else false
}
```

<br>



During development / testing, I used the simulator a lot and as simulator can't perform in-app purchase, I modified the code slightly like this so the screen will always show the full train time (easier for me to check)  :

```swift
func hasPurchasedPro(){
  
  return true
  
  // check Keychain to see if the purchase data exist or not
  // return true if it exist, else false
}
```

<br>



I would often comment and uncomment the line `return true` between development and production (submitting to App Store). And yes you guessed correct, one time I forgot to remove the `return true` statement and submitted the App to App Store 😅. The app got approved and my app has zero revenue for a week before I realized what I have done. 💸



"There must be a better way to do this", I thought, then I stumbled across **Launch arguments** / **Environment Variables** , which you can set certain variables which will only be injected during debug run (it has no effect when you archive the app and submit to App Store).





