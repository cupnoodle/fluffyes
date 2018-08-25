# Fix (Workaround) for Xcode playground stuck in Launching Simulator or Running Playground

[Jump to the solution if TL;DR](#workaround-solution)



Ever stuck in '**Launching Simulator**' or '**Running Playground**' when you fire up Playground to write some code? Despite restarting Xcode or even Mac multiple times, the problem still persists? I have seen many beginners give up on learning Swift because of this issue and this really sadden me, Apple should do better on this. Apple recommends beginner to use Playground and yet Playground is full of issues like this 🤦‍♂️.

![Launching](https://iosimage.s3.amazonaws.com/2018/26-fix-playground/launching.png)

![Running](https://iosimage.s3.amazonaws.com/2018/26-fix-playground/running.png)



I ran into this issue last week while trying out [NSPredicate examples](https://nspredicate.xyz) in Playground, it happened after an error during execution of code. 

![Error in playground](https://iosimage.s3.amazonaws.com/2018/26-fix-playground/error.png)



After the error appeared, no big deal, I changed the code and run it again, then the Playground got stuck in **Running**... 😱. Then I quit Xcode and open the Playground again, and the same issue persist 😭.



I tried restarting Xcode, Mac to no avail and almost attempt to reinstall Xcode. Then I noticed it stuck on "Launching Simulator" and thought it might have something to do with the iOS Simulator since restarting Xcode doesn't help 🤔.



When you create a Playground in Xcode, it will use iOS platform to run the code by default. To run it on iOS platform, the Playground will spin up an iOS Simulator in the background.

![Platform iOS](https://iosimage.s3.amazonaws.com/2018/26-fix-playground/platformiOS.png)



And it is this iOS Simulator thing that causes the stuck issues! Restarting Xcode doesn't work because the simulator process doesn't get restarted when you restart Xcode as it is a separate process.


![Core Simulator](https://iosimage.s3.amazonaws.com/2018/26-fix-playground/coreSimulator.png)



Somehow the CoreSimulator process become unstable after an error got raised in the Playground. It will get stuck after the error and remain stuck until you kill this process. That's the reason why you keep stuck on "Launching Simulator" or "Running.." even after restarting Xcode.




## Workaround Solution

The solution for this is to **switch to macOS platform** in the Playground, let the playground code run in your macOS environment directly instead of spinning up an iOS simulator process to run it.



First, ensure your Playground doesn't use any UIKit function (eg: UIView etc), and remove the top `import UIKit` line if it is there.


Then in the right pane of the Playground window, select '**macOS**' in **Playground Settings** -> **Platform** .

![select macOS platform](https://iosimage.s3.amazonaws.com/2018/26-fix-playground/selectMacOS.png)



I also suggest [switching to manually run](https://fluffy.es/xcode-playground-tips/#manual) instead of automatic run in the playground to reduce the CPU usage.



Now you should be able to run the Playground successfully, cheers! 🍻



The caveat of this solution is that you have to remove `import UIKit` and all UIKit related functions as macOS doesn't support UIKit.



## Workaround Solution if you need to use UIKit

If you are using UIKit functions in Playground, you have to use iOS Platform as UIKit is iOS-exclusive. 



The solution for solving the stuck issue then will be to quit Xcode, open '**Activity Monitor**' (In Spotlight, type 'Activity Monitor'), search for '**simulator**' ,highlight **com.apple.CoreSimulator** , and force quit it.



![quit simulator process](https://iosimage.s3.amazonaws.com/2018/26-fix-playground/quitSimulator.gif)



Then open Xcode again , it should work fine now. If you got any error during code execution in Playground, you will need to repeat these steps. I recommend [switching to manually run](https://fluffy.es/xcode-playground-tips/#manual) in Playground so that the Playground won't auto run the incomplete code while you are typing to prevent occurrence of error.







