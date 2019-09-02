# Replicating Facebook's Draggable Bottom Card using Auto Layout

There's a popular trend in recent iOS apps which when user tap a button, a card-like modal will appear from the bottom and user can drag to expand, contract, or dismiss it. Facebook and Slack app uses this UI design :

![examples](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/examples.png)



In this tutorial, we will analyze how to implement the bottom card modal, and try to replicate it using modal presentation (ie. `self.present(CardViewController)`) , Pan Gesture, Storyboard and Auto Layout. This post assume you have some experience working with [Auto Layout](https://autolayout.fluffy.es?ref=bottomcard). There's other ways to do this such as implementing a custom UIViewController Transition



The end result (with dimming and draggable bottom modal) will look like this : 

![small Demo](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/smallDemo.gif)





As the focus of this tutorial is implementing the bottom card modal view (not the Fake facebook status view shown above), I have created a starter project which contain the fake Facebook status view, you can [download the starter project here](https://fluffypublic.s3.amazonaws.com/starter_pack/bottomCard-starter.zip) .



The starter project contains a fake Facebook Status View Controller (Status View Controller) embedded inside a tab bar controller.

![starter project screenshot](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/starterScreenshot.png)



You can use your own view / view controller if you don't want to use the facebook status view design, as long as it has a UIButton that can activate the modal view controller on tap.



Build and run the starter project and you will see the fake Facebook Status View, tapping the "**😆 Tim Cook and 10 others**" button will call the `reactionListButtonTapped()` function inside **StatusViewController.swift** .



For the bottom modal card, we will create a view controller for it. Here's the big picture of the 7 main steps of this tutorial :



1. Create a new view controller (I named it as 'ReactionViewController', with a Storyboard ID of 'ReactionViewController' as well). Then the StatusViewController will present this new view controller modally, without animation.

   ![step 1](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/step1.png)



2. Add an UIImageView to the Reaction View Controller, and pin it to the top, bottom, leading and trailing edges. Note that the top constraint is towards "Superview's Top" (the absolute top of the screen), not the safe area, same goes to the bottom constraint.



![step 2](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/step2.png)



3. Take snapshot of the StatusViewController (Fake Facebook Status View), which is an UIImage, and set the image view in ReactionViewController to use this snapshot image, to give an illusion of transition.

![step 3](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/step3.png)



4. Add a gray UIView (with alpha 0.7 ish) on top of the image view, to serve as a dimmer view. Add a tap gesture recognizer on this gray view, when user tap on it, dismiss the ReactionViewController to go back to the StatusViewController.

![step4](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/step4.png)



5. Add a white UIView on top of the gray dimmer view, round the top left and top right corners. This is the card view we will use to display content. Create an IBOutlet for the top constraint from the card view top to the Safe Area Top.

![Step 5](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/step5.png)



6. Add a pan gesture recognizer on the ReactionViewController root view (i.e **self.view**) , and use code to change the top constraint constant value based on the distance panned by the user. i.e: when user drag finger up, the card view will expand upwards, when user drag finger down, the card view will contract downwards. 

![step 6](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/step6.png)



7. Add the handle view on top of the card view and also any additional content on the card as you like!

![step 7](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/step7.png)