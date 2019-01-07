# Why missing constraints appear and how to solve them

Making Sense of Auto Layout Series
[Chapter 1](https://fluffy.es/frame-vs-autolayout/) / [Chapter 2](https://fluffy.es/how-auto-layout-calculates-view-position-and-size/) / Chapter 3 / [Chapter 4](https://fluffy.es/why-conflicting-constraint-happen/) / [Chapter 5](https://fluffy.es/what-is-intrinsic-content-size/) / [Chapter 6](https://fluffy.es/what-is-constraint-priority/)

This is the third part of Making Sense of Auto Layout Series.  

Previously, we have explained [how Auto Layout calculates view position and size](https://fluffy.es/how-auto-layout-calculates-view-position-and-size/). In this post we will look into why Xcode shows you red lines when designing layout and how to solve them.

Xcode shows you red line in the Interface Builder (the place where you drag and drop view controller in storyboard) **when it doesn't know where it should place the view** or/and **what size is the view**.

Lets use a simple example :  

![Only Width and Height constraints defined](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/widthHeightConstraint.png)

The gray view has 2 constraints placed : 
1. Width is 240
2. Height is 120

The size of it is defined but there is red line surrounding it, it means that Auto Layout Engine doesn't know where to position it. We can troubleshoot what are the missing constraints by clicking the white arrow icon inside red circle button :  

![Show missing constraints](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/showError.png)

When you click on the missing constraints error, it will highlight the view that has missing constraint on Xcode.  

![Missing constraints for X, Y position](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/missingConstraintXY.png)

You will see that Xcode complain that it needs constraints that allow it to calculate the X position and Y position of the view. Remember Auto Layout Engine needs to know / able to calculate a view **X position**, **Y position**, **width** and **height** to render it correctly?

But there is no constraint that can let the Auto Layout Engine calculate the X position and Y position. Thats why Xcode surrounded the view with red lines to get your attention (oh you) to fix it.

There's many way to fix it (refer to examples in [this post](https://fluffy.es/how-auto-layout-calculates-view-position-and-size/)), the simplest way to fix it is adding a top and left constraint like this :  

![Add Top left constraint](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/addTopLeftConstraints.png)

After adding constraint, you will see those red line changed into blue line because Xcode now know where to position the view, yay!  
![Blue Line Yay](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/blueLineYay.png)

Auto Layout Engine now know that the view X position is 40, Y position is 40, width is 240 and height is 120. Hence it will render it nicely across devices like this :  

![Looks nice in both iPhone SE and 8 yay](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/solvedMissingConstraint.png)

### Summary
The example only used 1 view and I know that your app will surely have more than 1 view, but the cause of Xcode showing you red lines will always be that Xcode / Auto Layout Engine doesn't know **what size is your view** or **where to place your view**. 

[Always add enough constraints](https://fluffy.es/how-auto-layout-calculates-view-position-and-size/#checklist) to all the views so Xcode / Auto Layout Engine can know or able to calculate what size is your view and where to place your view.

In the next post, we will look into [how to solve conflicting constraints](https://fluffy.es/why-conflicting-constraint-happen/) (when Xcode show you red lines with numbers around your view).

<br>
<a href="https://www.getdrip.com/forms/448009646/submissions/new" target="_blank" data-drip-show-form="448009646" style="background-color:#4baa33; color:#fff; padding: 1rem; border-radius:4px;">Get Full Series PDF via Email</a>

### Homework exercise
I have prepared a Xcode project with some missing constraint, you can [download it here](https://www.dropbox.com/s/fi3hjg41kc9o13g/trafficLight.zip?dl=0).

This project contain only one view controller in the storyboard, like this :  
![Exercise](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/exercise.png)

There are three views and all of them have some constraints missing, your task is to **add those missing constraints** so that they look like this in iPhone SE, iPhone 8 and iPhone 8 plus :  

![Multiple screen size](https://iosimage.s3.amazonaws.com/2018/6-autolayout/multipleViewResult.png)

Tips :
You can view the layout for all of these devices (iPhone SE, 8 and 8 plus) in Xcode without running it in Simulator. You can add a preview view like this : 

![Open Assistant View](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/assistant.png)

![Select Preview View](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/assistant2.png)

In the preview view, click the **+** button on the bottom left to add device :  
![Add Devices](https://iosimage.s3.amazonaws.com/2018/7-missing-constraint/addDevice.png)

You can check the [answer of this exercise here](https://fluffy.es/how-auto-layout-calculates-view-position-and-size/#exercise), be sure to attempt it first before checking answer! 