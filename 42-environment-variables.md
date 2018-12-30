# Using launch arguments / environment variables to simulate in-app purchase purchased and etc


[TL;DR Jump to how to set launch arguments / environment variables](#howto)



If you have programmed in platform outside of iOS like Ruby , Python, Java etc, you might have used environment variable to store secrets. There's environment variable in iOS as well, these are injected to your app in runtime.



Here's how I learned to use launch argument / environment variable the hard way :



In my app [Komuter](https://komuter.app), users will need to purchase in-app purchase to view the full upcoming trains arrival time.

![purchase](https://iosimage.s3.amazonaws.com/2018/42-environment-variables/purchase.png)



In this screen, I call a function `hasPurchasedPro()` to check if the user has purchased in-app purchase. If the user has bought in-app purchase, the buy button and blurred text will be hidden, and the actual upcoming train time will be shown.



```swift
func hasPurchasedPro(){
  // check Keychain to see if the purchase data exist or not
  // return true if it exist, else false
}
```

<br>

(You can read more on [how to check if user has bought in-app purchase using Keychain here](https://fluffy.es/check-purchased-iap-using-keychain/))



During development / testing, I used the simulator a lot and as simulator can't perform in-app purchase, I modified the code slightly like this so the screen will always show the full train time (easier for me to check)  :

```swift
func hasPurchasedPro(){
  
  return true
  
  // check Keychain to see if the purchase data exist or not
  // return true if it exist, else false
}
```

<br>



I would often comment and uncomment the line `return true` between development and production (version that get submitted to App Store). And yes you have guessed correctly, one time I forgot to remove the `return true` statement and submitted it to the App to App Store 😅. The app got approved and my app has zero revenue for a week, until I realized what I have done, giving away the full version of my app to users for free during that week 😂💸. 



"There must be a better way to do this!", I thought, then I stumbled across **Launch arguments** / **Environment Variables** , which you can use to set some variables which will only be injected during debug run (it has no effect when you archive the app and submit it to the App Store).



<span id="howto"></span>

## How to set launch arguments / environment variables

To set launch arguments / environment variables, go to **Product** > **Scheme** > **Manage Schemes** .



![Manage scheme](https://iosimage.s3.amazonaws.com/2018/42-environment-variables/manageScheme.png)



Then double click the scheme with your app name : 

![double click](https://iosimage.s3.amazonaws.com/2018/42-environment-variables/doubleClick.png)



In the scheme window, select **Run (Debug)**  on the left side bar, and then select the **Arguments** tab on the right column. You will see two row **Arguments Passed On Launch** and **Environment Variables**. For the in-app purchase, we will set a launch argument by clicking the **+** button.

![arguments](https://iosimage.s3.amazonaws.com/2018/42-environment-variables/arguments.png)



Then we type in `BOUGHT` (can be any string you like) as one of the arguments.


Then in the **hasPurchasedPro** function, we can modify it to check for launch argument like this :

```swift
func hasPurchasedPro(){
  // if there is a 'BOUGHT' launch argument, simulate bought in-app purchase
  if ProcessInfo.processInfo.arguments.contains("BOUGHT") {
    return true
  }
  
  // check Keychain to see if the purchase data exist or not
  // return true if it exist, else false
  // ...
}
```

<br>



Build and run the app, we then see the app showing the unlocked screen 🙌 : 

![bought](https://iosimage.s3.amazonaws.com/2018/42-environment-variables/bought.png)



To revert back to the unpurchased state, simply uncheck the launch argument and build and run the app again : 

![uncheck](https://iosimage.s3.amazonaws.com/2018/42-environment-variables/uncheck.png)



Then we will see the screen with buy button and blurred text : 

![havent buy](https://iosimage.s3.amazonaws.com/2018/42-environment-variables/haventBuy.png)



With launch arguments, we can toggle different state easily, and no need to worry if we have accidentally shipped the full version of the app to the App Store 😅, as launch arguments / environment variables will not be included in Archive / Production version of the app.



The process is similar for environment variables, open the scheme window, and click the **+** button under **Environment Variables**. The difference is that you can set a key value pair for environment variable, whereas you can only set a string for launch argument.



![environment variable](https://iosimage.s3.amazonaws.com/2018/42-environment-variables/envvar.png)



Then in your code, you can check for the value of environment variables like this : 

```swift
switch ProcessInfo.processInfo.environment["level"] {
    case "member":
        // show member UI here
        print("member")
    case "moderator":
        // show moderator UI here
        print("moderator")
    case "administrator":
        // show administrator UI here
        print("administrator")
    default:
        // if the environment variable is not set or doesn't equal above values
        print("environment variable not set / doesn't equal values above")
}
```

<br><br><br>



If you find yourself commenting / uncommenting parts of code frequently, it might be a good idea to replace them with launch arguments / environment variables instead. You can easily check / uncheck them in scheme window instead of having to go to a specific file, uncomment the code and compile and run again.