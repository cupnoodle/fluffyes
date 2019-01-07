# How Auto Layout calculates view position and size

Making Sense of Auto Layout Series
[Chapter 1](https://fluffy.es/frame-vs-autolayout/) / Chapter 2 / [Chapter 3](https://fluffy.es/why-missing-constraints-appear/) / [Chapter 4](https://fluffy.es/why-conflicting-constraint-happen/) / [Chapter 5](https://fluffy.es/what-is-intrinsic-content-size/) / [Chapter 6](https://fluffy.es/what-is-constraint-priority/)

This is the second part of Making Sense of Auto Layout Series.　　
　　
Previously, we have explained the difference between [frame based layout and Auto layout with constraint](https://fluffy.es/frame-vs-autolayout/). In this post we will take a closer look on how Auto Layout calculates and position view.

For easier understanding, assume there is an Auto Layout Engine that takes input of **constraints** (the constraint you created in Xcode) and **app screen size** (depending on iOS device screen resolution, orientation).

[Here is the official screen resolution](https://developer.apple.com/library/content/documentation/DeviceInformation/Reference/iOSDeviceCompatibility/Displays/Displays.html#//apple_ref/doc/uid/TP40013599-CH108-SW2) of each iOS devices, for UI development we will use the **UIKit Size (Points)** column.

![Auto Layout Engine](https://iosimage.s3.amazonaws.com/2018/6-autolayout/engine.png)

In order for Auto Layout to render the position and size of a view correctly, Auto Layout engine **must know** or **able to calculate** the :

1. **x position** of a view
2. **y position** of a view
3. **width** of a view
4. **height** of a view

Defining constraint helps Auto Layout Engine to figure all of the above. Below I will show few examples of how Auto Layout calculate the position and size of a view using simple math (~grade school math).

We will discuss 4 examples and 1 exercise question in this post.

### Example 1

A straight forward example might look like this :  

![Straight Forward Auto Layout example](https://iosimage.s3.amazonaws.com/2018/6-autolayout/straightForward.png)

4 constraints are defined :
1. Distance from the screen left to the left of the blue view is 40 pt 
2. Distance from the screen top to the top of the blue view is 40 pt 
3. Width of the blue view is 160 pt
4. Height of the blue view is 100 pt

From the 1st constraint, we can deduce that the x position of the view is 40.
From the 2nd constraint, we can deduce that the y position of the view is 40.

Since x, y, width and height of the view is known, Auto Layout Engine will render it nicely across multiple screen size and orientation :  
![Staight Forward Auto Layout render result](https://iosimage.s3.amazonaws.com/2018/6-autolayout/straightForwardResult.png)

### Example 2

Similar to previous example, we use back the same width and height but this time we define a right and bottom constraint like this : 
![Bottom right constraint](https://iosimage.s3.amazonaws.com/2018/6-autolayout/bottomRightConstraint.png)

4 constraints are defined :
1. Distance from the screen right to the right of the blue view is 40 pt 
2. Distance from the screen bottom to the bottom of the blue view is 40 pt 
3. Width of the blue view is 160 pt
4. Height of the blue view is 100 pt

To calculate **x position** of the blue view, Auto Layout Engine will use the screen width. Calculation on an iPhone SE (screen width 320 pt) will look like this : 

```
screenWidth = 320
distanceFromRight = 40
viewWidth = 160

// x position of the blue view
x = screenWidth - distanceFromRight - viewWidth
x = 320 - 40 - 160
x = 120
```

<br>

To calculate **y position** of the blue view, Auto Layout Engine will use the screen height. Calculation on an iPhone SE (screen height 568 pt) will look like this : 

```
screenHeight = 568
distanceFromBottom = 40
viewHeight = 100

// y position of the blue view
y = screenHeight - distanceFromBottom - viewHeight
y = 568 - 40 - 100
y = 428
```

<br>

Using the same calculation formula, iPhone 8 (screen width 375, height 667) will calculate the x position as 175, y postion as 527.

Now x, y, width and height of the view is known, Auto Layout Engine will render it nicely across multiple screen size and orientation :

![Bottom Right constraint result](https://iosimage.s3.amazonaws.com/2018/6-autolayout/bottomRightConstraintResult.png)

### Example 3

In this example, there is no width or height constraint defined, instead we will set the top, left, right and bottom constraint from the blue view to the screen : 

![Four side constraint](https://iosimage.s3.amazonaws.com/2018/6-autolayout/fourSideConstraint.png)

4 constraints are defined :
1. Distance from the screen top to the top of the blue view is 60 pt 
2. Distance from the screen left to the left of the blue view is 40 pt 
3. Distance from the screen right to the right of the blue view is 40 pt 
4. Distance from the screen bottom to the bottom of the blue view is 60 pt 

From the 1st constraint, we can deduce that the y position of the view is 60.
From the 2nd constraint, we can deduce that the x position of the view is 40.

To calculate **width** of the blue view, Auto Layout Engine will use the screen width. Calculation on an iPhone SE (screen width 320 pt) will look like this : 

```
screenWidth = 320
distanceFromLeft = 40
distanceFromRight = 40

// width of the blue view
viewWidth = screenWidth - distanceFromLeft - distanceFromRight
viewWidth = 320 - 40 - 40
viewWidth = 240
```

<br>

To calculate **height** of the blue view, Auto Layout Engine will use the screen height. Calculation on an iPhone SE (screen height 568 pt) will look like this : 

```
screenHeight = 568
distanceFromTop = 60
distanceFromBottom = 60

// height of the blue view
viewHeight = screenHeight - distanceFromTop - distanceFromBottom
viewHeight = 568 - 60 - 60
viewHeight = 448
```

<br>

Using the same calculation formula, iPhone 8 (screen width 375, height 667) will calculate the width as 295, height as 547.

Now x, y, width and height of the view is known, Auto Layout Engine will render it nicely across multiple screen size and orientation :

![Four side constraint result](https://iosimage.s3.amazonaws.com/2018/6-autolayout/fourSideConstraintResult.png)

### Example 4

This example will be a bit more difficult to calculate than previous ones, the constraints are defined like this : 

![Center horizontal constraint](https://iosimage.s3.amazonaws.com/2018/6-autolayout/centerHorizontalConstraint.png)

4 constraints are defined :
1. Distance from the screen top to the top of the blue view is 50 pt 
2. Distance from the screen left to the left of the blue view is 50 pt 
3. Height of the blue view is 160 pt 
4. Blue view is horizontally centered to the screen 

From the 1st constraint, we can deduce that the y position of the view is 50.
From the 2nd constraint, we can deduce that the x position of the view is 50.

The remaining step is to calculate the width of the blue view since height is given. Since blue view is horizontally centered, it means that the left half of the width is same as the right half, like this :  

![Half width](https://iosimage.s3.amazonaws.com/2018/6-autolayout/halfWidth.png)

From the above diagram, we can deduce that :  
```
50 + half of blue view width = half of screen width
```

<br>

To calculate **width** of the blue view, Auto Layout Engine will use the screen width. Calculation on an iPhone SE (screen width 320 pt) will look like this :  
```
screenWidth = 320
distanceFromLeft = 50

distanceFromLeft + (viewWidth / 2 ) = screenWidth / 2 
50 + (viewWidth / 2) = 320 / 2
50 + (viewWidth / 2) = 160
viewWidth / 2 = 160 - 50
viewWidth / 2 = 110
viewWidth = 110 * 2
viewWidth = 220
```

<br>

Using the same calculation formula, iPhone 8 (screen width 375, height 667) will calculate the width as 275.

Now x, y, width and height of the view is known, Auto Layout Engine will render it nicely across multiple screen size and orientation :

![Half width result](https://iosimage.s3.amazonaws.com/2018/6-autolayout/halfWidthResult.png)

This example is useful for cases where you want to center a view with equal left spacing and right spacing. Instead of setting left space constraint and right space constraint manually to the same number, you can just set the left space constraint and set a horizontally center constraint on the view. 

### Homework exercise - Multiple View
<span id="exercise"></span>
We have discussed various examples and now its time to do some hands on calculation 📝! Although previous examples only use one view, the same concept and calculation still applies for multiple view.

Here is the exercise question : 

Calculate the x position, y position, width and height of red, yellow and green view. Assuming iPhone SE screen size is used (320pt x 568 pt)

The constraints are defined as follow :  
![Three view with constraint](https://iosimage.s3.amazonaws.com/2018/6-autolayout/multipleView.png)

1. Distance from the screen top to the top of red view is 30 pt
2. Distance from the screen left to the left of red view is 30 pt
3. Height of red view is 120 pt
4. Distance from bottom of red view to top of yellow view is 30 pt
5. Distance from the screen left to the left of yellow view is 30 pt
6. Height of yellow view is 120 pt
7. Distance from bottom of yellow view to top of green view is 30 pt
8. Distance from the screen left to the left of green view is 30 pt
9. Height of green view is 120 pt
10. Red view is horizontally centered on the screen
11. Yellow view is horizontally centered on the screen
12. Green view is horizontally centered on the screen

The answer will be at the end of this post, be sure to try it before looking at the answer! 🤔
Hint : Start calculating from the view at the top first then proceed to bottom.

### Summary
The takeaway of this post is that if Auto Layout Engine can calculate all of the views position and size using the given constraints, it will render them nicely. If there is not enough constraint, Auto Layout Engine can't calculate position/size of some view, it will attempt to 'guess' their position/size and thats the reason why you see some layout going haywire when constraints are not defined correctly. 

Here is the checklist to ensure Auto Layout Engine can render the view correctly.  

<div id="checklist" style="background-color: #FFF; padding: 1em; border: 1px solid #666; border-radius: 5px;">

Does the constraints defined allow Auto Layout Engine to know / calculate : 
1. **x position** of all view ✅
2. **y position** of all view ✅
3. **width** of all view ✅
4. **height** of all view ✅
  </div>
  <br>
  One quick way to determine if Auto Layout Engine is able to calculate them (position and size) is to calculate it yourself, by using pen and paper. If you can't deduce them using math, Auto Layout Engine can't either. Once you get a hang of it, you won't need to calculate these by hand, as long as you know how to define constraints that enable Auto Layout Engine to calculate these, your layout will look great on every device.  


<br>
<br>

In the next post, we will look into [how to solve missing constraint](https://fluffy.es/why-missing-constraints-appear) (when Xcode show you red lines around your view.)  

<br>
<a href="https://www.getdrip.com/forms/448009646/submissions/new" target="_blank" data-drip-show-form="448009646" style="background-color:#4baa33; color:#fff; padding: 1rem; border-radius:4px;">Get Full Series PDF via Email</a>


## Exercise Answer
On iPhone SE screen :  
![Multiple View Answer](https://iosimage.s3.amazonaws.com/2018/6-autolayout/multipleView2.png)

<span style="color: #FC5F45;">Red View</span> : 
1. X position is 30
2. Y position is 30
3. Width is 260
4. Height is 120

<span style="color: #FEDA68;">Yellow View</span> :
1. X position is 30
2. Y position is 180
3. Width is 260
4. Height is 120

<span style="color: #5EB863;">Green View</span> : 
1. X position is 30
2. Y position is 330
3. Width is 260
4. Height is 120

This is how it looks like in multiple screen sizes : 
![Multiple view result](https://iosimage.s3.amazonaws.com/2018/6-autolayout/multipleViewResult.png)