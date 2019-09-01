# Replicating Facebook's Draggable Bottom Card

There's a popular trend in recent iOS apps which when user tap a button, a card-like modal will appear from the bottom and user can drag to expand, contract, or dismiss it. Facebook and Slack app uses this UI design :

![examples](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/examples.png)



In this tutorial, we will analyze how to implement the bottom card modal, and try to replicate it using modal presentation (ie. `self.present(CardViewController)`) , Pan Gesture, Storyboard and Auto Layout. This post assume you have some experience working with [Auto Layout](https://fluffy.es/making-sense-of-auto-layout/). There's other ways to do this such as implementing a custom UIViewController Transition



The end result (with dimming and draggable bottom modal) will look like this : 

![small Demo](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/smallDemo.gif)





As the focus of this tutorial is implementing the bottom card modal view (not the Fake facebook status view shown above), I have created a starter project which contain the fake Facebook status view, you can [download the starter project here](https://fluffypublic.s3.amazonaws.com/starter_pack/bottomCard-starter.zip) .



The starter project contains a fake Facebook Status View Controller (Status View Controller) embedded inside a tab bar controller.

![starter project screenshot](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/starterScreenshot.png)



You can use your own view / view controller if you don't want to use the facebook status view design, as long as it has a UIButton that can activate the modal view controller on tap.



Build and run the starter project and you will see the fake Facebook Status View, tapping the "**😆 Tim Cook and 10 others**" button will call the `reactionListButtonTapped()` function inside **StatusViewController.swift** .



For the bottom modal card, we will create a view controller for it. Here's the big picture of the steps of this tutorial :







