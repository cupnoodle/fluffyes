# Tips for using Xcode Playground

> Xcode Playground might be actually useful if it doesn't crash like every 15 minutes and suck up my CPU usage

I was excited when Apple first announced Playground. After updating to the latest Xcode (at that time), I hurried to type some code in it and amazed by its instant output, then it crashed like after 10 minutes, wtf Apple?!

Playground is a great place to test out small chunk of code as you dont need simulator to see the output and the output is almost immediate. The problem with it is that it crashes really often and causes your mac fan to spin loudly.

Here are some tips to reduce Playground crashes / CPU usage, improve productivity and using Playground for testing JSON.

1. [Set to manually run](#manual)
2. [Map shortcut key to execute playground](#shortcut)
3. [Set to indefinite execution for asynchronous function](#indefinite)

Xcode playground stuck in Launching Simulator or Running? [Read the workaround fix here](https://fluffy.es/fix-playground-stuck/) .

<span id="manual"></span>
## Set to manually run
One reason Playground eat up CPU usage is because it will **auto build and run your code every few seconds / keystroke**. To reduce CPU usage, simply turn off auto run by selecting **Manually Run** like this : 
![Set to manually run](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/manually_run.png)

Then you can click the same button (▶️) again to run the code when you have finished typing. This should reduce crash frequency in Playground and also lower the CPU usage.

<span id="shortcut"></span>
## Map shortcut key to execute playground
Don't like to move mouse around and prefer to use keyboard shortcuts? By default, Xcode doesn't have a shortcut key to execute playground code, you can assign a shortcut key to it like this :

1. Select **Xcode** > **Preferences**
  ![Xcode Preferences](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/xcode_preferences.png)

2. Select **Key Bindings**, search for **Execute Playground** and input the shortcut key you want
  ![Key Bindings](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/key_bindings.png)

Now you can run the code in Playground by pressing the shortcut key you have just set.

### Map ⌘R key to execute playground
You might be used to press <kbd>⌘</kbd><kbd>R</kbd> to run Xcode project code and want to use back the same key to execute Playground code as well.

Sadly, an error will occur when you try to map ⌘R to Execute Playground as this key has already been mapped to "Run" : 
![Clash With Run](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/clash_with_run.png)

<br>
​    
You can get around this error by using the macOS system keyboard shortcut setting to override it.

1. Open **System Preferences** > **Keyboard**
  ![Select Keyboard](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/select_keyboard.png)

2. Select **Shortcuts**
  ![Select shortcuts](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/select_shortcuts.png)

3. Select **App Shortcuts**, then click **"+"**. Select `Xcode` for the Application, type `Execute Playground` for the Menu Title (must follow the text exactly, else it might not work), then input `⌘R` as the keyboard shortcut.
  ![Add app shortcut](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/add_app_shorcut.png)


You should see the 'Execute Playground' shortcut key created for Xcode like this :
![Xcode Execute Playgroun shortcut](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/playground_shortcut.png)

Now you can execute the Playground code by pressing <kbd>⌘</kbd><kbd>R</kbd> just like in normal Xcode Project!

As of Xcode 9, assigning ⌘R to ` Execute Playground` won't conflict with `Run` as typical Xcode project dont have the Playground execute button and Playground dont have the run button of Xcode Project. You can stop worry about key conflict (for now).

<span id="indefinite"></span>
## Set to indefinite execution for asynchronous function
I often use Playground to test JSON response received from a web API, it's really fast to test without having to open simulator.

Let's say we have a URLSession code to get Car JSON data like this in Playground:
![Get Car API](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/get_car.png)

After executing it, you will notice that the response data is not shown in the console log, but when you try it in a Xcode project, it works! 😫

This is because by default, Playground will stop running when it reach the last line of the code. After executing the last line `task.resume()`, Playground will stop running immediately, even before the JSON response is received, hence the completion handler of the URLSession will not get executed and the output didnt show.

Here's a visual explanation:
![Visual explanation](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/explanation.png)

<br>

To make Playground wait for the JSON response to return, we can add some code to tell the Playground to keep on running until we stop them :
```swift
import UIKit
import Foundation
// Import this to modify Playground setting
import PlaygroundSupport

// Tell playground to keep on running until we click the stop button
PlaygroundPage.current.needsIndefiniteExecution = true

struct Car: Decodable {
	let name: String
	let horsepower: Int
}

...
```

<br>

Now execute the Playground code again and Playground will keep on running,  wait for the JSON response to arrive and parse it: 
![Yay JSON arrived](https://iosimage.s3.amazonaws.com/2018/16-playground-tips/yay_output.png)

Remember to click the Stop button when you have retrieved the JSON response to conserve CPU usage.

Setting `PlaygroundPage.current.needsIndefiniteExecution = true` is really helpful when you want to run asynchronous function which the result will not return immediately in the Playground.


### Using code to stop the playground execution
_Update: 15 May 2018_

[Scott](http://scotteg.com/) has commented about using `PlaygroundPage.current.finishExecution()` to stop the execution of the playground programmatically so that we don't have to remember to click the Stop button. Thanks Scott!

We can instruct the Playground to stop executing by adding `PlaygroundPage.current.finishExecution()` inside the completion handler. This will cause the playground to stop running after parsing the received json or after encountering error while parsing json.

```swift
// Get car json data
let task = URLSession.shared.dataTask(with: url, completionHandler: { (data, response, error) in
    guard let data = data else { return }
    do {
        let decoder = JSONDecoder()
        let car = try decoder.decode(Car.self, from: data)
        print("car name is \(car.name)")
        print("car horsepower is \(car.horsepower)")
			
    } catch let error {
        print("Failed to decode JSON:", error)
    }
	
    // stop the playground execution
    PlaygroundPage.current.finishExecution()
})
```

<br>



<div class="post-subscribe">
  <div class="post-subscribe-left">
     <h4 data-drip-attribute="headline">Want to level up your iOS development skills?</h4>
    <span style="font-size:0.9rem;"> 
            Sign up below and I'll send you articles just like this about iOS development to your inbox every two week-ish.
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/295054774/submissions" method="post" data-drip-embedded-form="295054774">
                <div style="margin-bottom: 0.5rem;">
                    <label for="drip-firstname">Name<span style="color:#952B45;">*</span></label><br />
                    <input type="text" id="drip-firstname" name="fields[firstname]" value="" />
                </div>
                <div>
                    <label for="drip-email">Email Address<span style="color:#952B45;">*</span></label><br />
                    <input type="email" id="drip-email" name="fields[email]" value="" />
                </div>
              <div>
                <br>
                <input type="submit" value="Sign me up!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips.<br> Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>