# 3 steps to speed up storyboard

When you are working on storyboard with many view controllers, especially with [IBDesignable](https://nshipster.com/ibinspectable-ibdesignable/) UI, it can take quite a long while to open storyboard, and then you hear your Mac fan spins like it is about to take off 🚀 😂.



![hot cpu](https://iosimage.s3.amazonaws.com/2020/78-speed-up-storyboard/hot.png)



Here's 3 steps you can take to speed up the storyboard.



## 1. Remove the use of IBDesignable View if possible

IBDesignable is great for showing custom view attributes lilke cornerRadius, shadow on the storyboard without having to build and run the app. However Xcode seems to use a lot of resource on rendering IBDesignable, and sometimes it will automatically build when you are just trying to open Storyboard.


If possible, try to remove IBDesignable view on your storyboard's view controller, and initialize it in your code instead.




## 2. Uncheck automatically refresh views

If it is not possible to remove IBDesignable View, and you find that Xcode keeps building automatically after each key press, you can try uncheck "Automatically Refresh Views".



Open your storyboard (make sure it is in the active tab), then **uncheck** Editor > Automatically Refresh Views.

![uncheck](https://iosimage.s3.amazonaws.com/2020/78-speed-up-storyboard/uncheck.png)



This will stop Xcode from automatically building the storyboard every time you change your code.



You can enable it back as needed when you are done with your code, to see the updated visual preview.



Alternatively, you can keep it off, and click "Refresh All Views" in the Editor menu, to refresh the view manually.

![refresh all views](https://iosimage.s3.amazonaws.com/2020/78-speed-up-storyboard/refreshAllViews.png)



## 3. Use multiple storyboard files and storyboard references

If you stuff all view controllers into one storyboard, it will get slow eventually. I suggest one storyboard should have less than 15 view controllers.



You can select a few related view controllers, usually inside the same navigation controller, then select **Editor** > **Refactor to Storyboard** .



![refactor](https://iosimage.s3.amazonaws.com/2020/78-speed-up-storyboard/refactor.png)



Xcode will then create a new storyboard file for you, and also a storyboard reference from the previous controller. The storyboard reference will tell the previous controller to continue the segue in the newly created storyboard.



![storyboard reference demo](https://iosimage.s3.amazonaws.com/2020/78-speed-up-storyboard/reference.gif)



Usually I will refactor view controllers inside the same navigation controller or tab bar controller into a new storyboard, and name the storyboard based on the tab name or the flow name of the navigation controller (eg: user registration ).



With multiple storyboards, different dev in the team can work on different storyboard and avoid merge conflict when merging different feature branch.



Storyboard is just a XML file underneath, parsing XML files and generating layout from it can be time consuming as the XML file grows larger (with addition of more views). It is best to keep each storyboard files small.



Or alternatively you can say "screw it", and go for [coding the user interface programmatically](https://fluffy.es/intro-to-creating-ui-in-code-1/) 😂. For small personal projects I would still use Storyboard / XIBs as it is easier to visualize the design, and size classes.



















