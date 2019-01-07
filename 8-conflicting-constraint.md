# Why conflicting constraint happen and how to solve them

Making Sense of Auto Layout Series
[Chapter 1](https://fluffy.es/frame-vs-autolayout/) / [Chapter 2](https://fluffy.es/how-auto-layout-calculates-view-position-and-size/) / [Chapter 3](https://fluffy.es/why-missing-constraints-appear/) / Chapter 4 / [Chapter 5](https://fluffy.es/what-is-intrinsic-content-size/) / [Chapter 6](https://fluffy.es/what-is-constraint-priority/)

This is the fourth part of Making Sense of Auto Layout Series.  

Previously, we have explained [why missing constraints appear and how to solve them](https://fluffy.es/why-missing-constraints-appear/). In this post we will look into why Xcode shows you red lines with numbers when designing layout and how to solve them.

Ever experience that you finally got rid of all the red lines and it looks good on your device, but when you view it on another device in simulator, it look totally off?

This is probably because you have conflicting constraint in the layout, we will use a simple example below. The example contain a vertically centered button.

![Looks good on iPhone X](https://iosimage.s3.amazonaws.com/2018/8-conflicting-constraint/looksNice.png)

The constraints are :  
1. Width of the button equals 214 pt
2. Distance from the screen left to the left of the button is 80 pt
3. Distance from the screen right to the right of the button is 81 pt
4. Distance from the screen top to the top of the button is 352 pt

It looks good and all the lines are blue, yay! Enough constraints are defined and Xcode is able to calculate the x position, y position, width and height of the button.

It looks good on iPhone X, but once you switch the view to iPhone SE, there's red lines everywhere with numbers 😱 (this mean that theres conflicting constraints), and the button text got squeezed :  

![Oh no when viewed in iPhone SE](https://iosimage.s3.amazonaws.com/2018/8-conflicting-constraint/ohNo.png)

Similar to solving missing constraint, we can troubleshoot conflicting constraints by clicking white arrow icon inside red circle button :  

![Layout troubleshoot icon](https://iosimage.s3.amazonaws.com/2018/8-conflicting-constraint/redArrow.png)

And we will see that Xcode complains to us that there's conflicting constraints :  
![Conflicting constraints](https://iosimage.s3.amazonaws.com/2018/8-conflicting-constraint/conflictingConstraints.png)

Hmmm... so whats with the conflicting constraints? Remember [how Auto Layout calculate size of a view](https://fluffy.es/how-auto-layout-calculates-view-position-and-size/)? We will first discuss why the layout works fine on iPhone X.


### Why it works fine on iPhone X
Since iPhone X has a screen width of 375 point,

From the leading and trailing constraints, Auto Layout Engine can deduce that the width of the button with this calculation :  
```
ScreenWidth = 375
LeadingConstraint = 80
TrailingConstraint = 81
widthOfButton = ScreenWidth - LeadingConstraint - TrailingConstraint
widthOfButton = 375 - 80 - 81
widthOfButton =  214
```

<br>

So the width of button is 214 by calculation of leading and trailing constraints we have defined. But remember we also explicitly defined the width of the button to 214.

```
// explicitly defined width constraint
widthOfButton = 214
```

Since the explicitly defined width is equal to the width derived from using leading and trailing constraints, Xcode doesn't complain.

### Why it doesn't work on iPhone SE (and also other screen width)

Well iPhone SE has a screen width of 320 point,

Using the same calculation with leading and trailing constraints :  
```
ScreenWidth = 320
LeadingConstraint = 80
TrailingConstraint = 81
widthOfButton = ScreenWidth - LeadingConstraint - TrailingConstraint
widthOfButton = 320 - 80 - 81
widthOfButton =  159
```

<br>

We calculated that the width of the button should be 159 points using the leading and trailing constraint.

But since we have another constraint explicitly mentioned the width of button to 214, Xcode got confused (use 159 as width or 214? they are conflicting 😫) thus showing you red lines with numbers.

When you run the app, you will see some error message in the console like this :  

![Error console](https://iosimage.s3.amazonaws.com/2018/8-conflicting-constraint/errorConsole.png)

Auto Layout Engine can't satisfy all of the constraints as it is impossible to set the width of the button to 159 and 214 at the same time. Thus to render the button on screen, Auto Layout Engine will break (ignore) one of the constraints ^[Auto Layout Engine doesn't randomly choose constraint to break, it will choose with a certain order, but I don't really know whats the exact order], in this case, the explicit width 214 constraint will be ignored. After breaking (ignoring) the width 214 constraint, Auto Layout Engine will use the 159 as width of the button. As the width is too short, the text in the button got squeezed.

When there are several conflicting constraints while the app is running, Auto Layout Engine will start to break constraint one by one with a certain order until there is no conflict in the layout calculation (size and position) then only render the layout.

### Fix
To resolve conflicting constraint, click on the red icon with white dot and select the constraint to delete. In this example, we will delete the explicit width 214 constraint so that the width of the button can be dynamic depending on the screen width.

![Delete conflicting constraint](https://iosimage.s3.amazonaws.com/2018/8-conflicting-constraint/deleteConflictingConstraint.png)

After deleting the conflicting constraint, Xcode doesn't confuse anymore and show us blue lines on the iPhone SE screen (and other screen size too) , yay!

![Blue Line Yay](https://iosimage.s3.amazonaws.com/2018/8-conflicting-constraint/blueLineYay.png)

Ahh the text still got squeezed because the width from calculation is too short, we need to reduce the distance from screen left and screen right. This is straightforward so I won't further explain it here.

### Summary
Conflicting constraints usually happen when you have defined too many constraint on an UI element causing Auto Layout Engine unable to satisfy all of them. 

I noticed quite some beginner iOS Developer is prone to making conflicting constraints because they usually design UI for only one device type (usually the iPhone model they are using) in the Interface Builder during development. They might unconsciously place excess constraints and Xcode shows blue lines on the device they are designed to. But when they have finished developing the app and release it to friends or beta tester, the layout problem arise as different screen size is used.

To prevent accidentally placing conflicting constraint, be sure to switch between different iPhone model when placing a new UI element / deleting an existing UI element in the Interface Builder.

![Device list](https://iosimage.s3.amazonaws.com/2018/8-conflicting-constraint/deviceList.png)

In the next post, we will look into [what is intrinsic content size and the usefulness of it](https://fluffy.es/what-is-intrinsic-content-size/).

<br>
<a href="https://www.getdrip.com/forms/448009646/submissions/new" target="_blank" data-drip-show-form="448009646" style="background-color:#4baa33; color:#fff; padding: 1rem; border-radius:4px;">Get Full Series PDF via Email</a>