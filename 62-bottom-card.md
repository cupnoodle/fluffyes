# Replicating Facebook's Draggable Bottom Card using Auto Layout - Part 1/2

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



There's all to it! Most of the complex coding is on step 6 (manipulating the top constraint constant value based on the pan gesture).



Without further ado, let's start the tutorial!



## Create ReactionViewController with backing Image View and gray dimmer View

Open the starter project, create a new ViewController file , and name it as "**ReactionViewController**". Next, open the Main.storyboard, drag a new view controller into the storyboard, set its class to ReactionViewController and set its Storyboard ID to 'ReactionViewController' (same name as the class).



![drag a new VC](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/dragNewVC.png)



Next, open "StatusViewController.swift", find the function **@IBAction func reactionListButtonTapped()**. This function is fired when the "Tim Cook and 10 others" reaction button on the status view is tapped.



We want to present the ReactionViewController modally when this button is tapped, add the following code inside the function : 

```swift
@IBAction func reactionListButtonTapped(_ sender: UIButton) {
  guard let reactionVC = storyboard?.instantiateViewController(withIdentifier: "ReactionViewController") 
  as? ReactionViewController else {
      assertionFailure("No view controller ID ReactionViewController in storyboard")
      return
  }

  // present the view controller modally without animation
  self.present(reactionVC, animated: false, completion: nil)
}
```

<br>

If we present a view controller modally with **animated** set to true, iOS will use a default animation of moving the new controller from the bottom edge to the top. We don't want this animation as we are going to replace it with our own card modal animation.



Build and run the project, when you tap on the button, you will see a blank view controller appear immediately like this : 

![present blank VC](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/presentBlankVC.gif)



As the blank view controller looks very different than the previous view controller, this might give user a sudden shock. To smoothen the transition, we will take a snapshot of the previous view controller, and put the snapshot image as the background of the new view controller (ReactionViewController), so that user won't notice that there is a transition of view controllers.



To capture an image snapshot of a view, we will use **UIGraphicsImageRenderer** class provided by Apple to render the UIView as an image. Create a new Swift file, name it as "UIView+Snapshot.swift", then add an extension to the UIView class like this :

```swift
// UIView+Snapshot.swift

import UIKit

extension UIView  {
    // render the view within the view's bounds, then capture it as image
  func asImage() -> UIImage {
    let renderer = UIGraphicsImageRenderer(bounds: bounds)
    return renderer.image({ rendererContext in
        layer.render(in: rendererContext.cgContext)
    })
  }
}
```

<br>



Next, we will place an UIImageView on the ReactionViewController, this image view will use the image snapshot we captured earlier on. Drag an UIImageView into ReactionViewController, then set top, bottom, leading and trailing constraint to 0 like this :

 

![constraints](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/imageviewConstraints.png)



By default Xcode will set the top and bottom constraint to the Safe Area (the rectangular area not hidden by the notch and rounded corner of the phone screen). We want the image view to cover the whole screen including the rounded corner of the phone screen, to do this, open the Size inspector tab to list all constraints for the image view.



Double click the Top constraint:

![top constraint](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/imageViewTopConstraint.png)



In the First Item or Second Item, change the **SafeArea.Top** to **SuperView.Top**, and after changing, set the constant to **0**.

![superview top](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/superviewTop.png)



Do the same for the bottom constraint, change the **SafeArea.Bottom** to **SuperView.Bottom**, and set the constant to **0**.



![superview bottom](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/superviewBottom.png)



After setting both top and bottom constraint to compare against superview, the UIImageView should look like this in the view controller : 



![image view whole](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/imageViewWhole.png)



Next, create an IBOutlet for this image view, an declare an UIImage variable to store the snapshot image, in ReactionViewController.swift : 

```swift
// ReactionViewController.swift

class ReactionViewController: UIViewController {

  @IBOutlet weak var backingImageView: UIImageView!
    
  // to store backing (snapshot) image
  var backingImage: UIImage?
  
  override func viewDidLoad() {
    super.viewDidLoad()

    // update the backing image view
    backingImageView.image = backingImage
  }  
}
```

Don't forget to update the imageview's image in viewDidLoad as well.



Now in the StatusViewController, we can capture the snapshot of its view right when the button is tapped, instantiate the ReactionViewController, and pass the snapshot image to the **backingImage** variable.



```swift
// StatusViewController.swift

@IBAction func reactionListButtonTapped(_ sender: UIButton) {
  guard let reactionVC = storyboard?.instantiateViewController(withIdentifier: "ReactionViewController")
    as? ReactionViewController else {
      
    assertionFailure("No view controller ID ReactionViewController in storyboard")
    return
  }

  // take a snapshot of current view and set it as backingImage
  reactionVC.backingImage = self.view.asImage()

  // present the view controller modally without animation
  self.present(reactionVC, animated: false, completion: nil)
}
```

<br>



Build and run the project, tap the button and you will see that nothing has changed after tapping button, it means that the snapshot image is being shown correctly! The ReactionViewController has presented modally and its backing image view is using the snapshot captured on StatusViewController.



Upon closer inspection, the snapshotted image didn't have tab bar?! 😱

![where's tab bar](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/wheresTabBar.png)



The snapshot from **self.view.asImage()** in **StatusViewController** doesn't contain tab bar, this is because StatusViewController is embedded inside Tab Bar Controller, hence calling capture on itself doesn't capture UI outside of its bounds. To fix this, we should call **self.tabBarController?.view.asImage()** instead, to capture the whole view including tab bar.



```swift
// StatusViewController.swift
@IBAction func reactionListButtonTapped(_ sender: UIButton) {
  guard let reactionVC = storyboard?.instantiateViewController(withIdentifier: "ReactionViewController") as? ReactionViewController else {
    assertionFailure("No view controller ID ReactionViewController in storyboard")
    return
  }
  
  // capture the whole view including tab bar
  reactionVC.backingImage = self.tabBarController?.view.asImage()
  self.present(reactionVC, animated: false, completion: nil)
}
```

<br>



Build and run the project again, this time the tab bar is there after pressing the button! But wait, the button seems faded out?



![faded out button](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/buttonFadedOut.png)



On default UIKit setting, when a button is being tapped, it will fade out for a while before restoring back the original opacity. As the snapshot is captured immediately right after user tap the button, the button is still in its faded out state during the capture. 



One of the way to solve this is to delay the capture of the snapshot and subsequently the modal presentation, we can use DispatchQueue to delay the execution of screen capture like this : 

```swift
// StatusViewController.swift

@IBAction func reactionListButtonTapped(_ sender: UIButton) {
  guard let reactionVC = storyboard?.instantiateViewController(withIdentifier: "ReactionViewController")
    as? ReactionViewController else {
      assertionFailure("No view controller ID ReactionViewController in storyboard")
      return
  }
  
  // Delay the capture of snapshot by 0.1 seconds
  DispatchQueue.main.asyncAfter(deadline: .now() + 0.1 , execute: {
    // take a snapshot of current view and set it as backingImage
    reactionVC.backingImage = self.tabBarController?.view.asImage()
    
    // present the view controller modally without animation
    self.present(reactionVC, animated: false, completion: nil)
  })
}
```

<br>



Build and run the project, now we have implemented an illusion which user feel like they are still on the same view controller despite ReactionViewController has been presented modally!



Next, we will put a dimmer view on top of the backing image view in ReactionViewController. Apply the same top, bottom (to superview instead of safe area), leading and trailing constraints for the dimmer view as well. 



Set the background color of the dimmer view to **Dark Gray** (or any other darker color you want), then set the alpha to **0.7** . Your storyboard should look like this now :

![dimmer view prop](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/dimmerViewProp.png)



Next, create an IBOutlet for this dimmer view in ReactionViewController, we can name it as **dimmerView** .

```swift
// ReactionViewController.swift

class ReactionViewController: UIViewController {

  @IBOutlet weak var backingImageView: UIImageView!
  @IBOutlet weak var dimmerView: UIView!

  // to store backing (snapshot) image
  var backingImage: UIImage?
  
  // ...
}
```



Now we have the dimmer view, it's time to put the actual card view on the ReactionViewController. Drag a new UIView onto the view controller, place it on top of the dimmer view, then add the constraints like this : 

![card view constraints](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/cardViewConstraints.png)



The bottom constraint of the card view should be relative to the **Superview bottom** (same as previous dimmer view), but the **top constraint** of the card view should be relative to the **SafeArea top**, and with a constant number (I have set it to 30) : 



![cardview constraint list](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/cardViewConstraintList.png)



 Next, create an IBOutlet for the card view and the top constraint of the card view, you can name the IBOutlet for the card view as **cardView**. 



To create an IBOutlet for the top constraint, double click the top constraint in the constraint list, then Xcode will highlight the constraint on the Document Outline, then press and hold **Control** and drag it to the view controller's code.



![double click top constraint](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/doubleClickTopConstraint.png)



![top constraint IBOutlet](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/topconstraintIBOutlet.png)





I have named the top constraint IBOutlet as **cardViewTopConstraint** . Your ReactionViewController.swift file should look like this now :



```swift
// ReactionViewController
class ReactionViewController: UIViewController {

    @IBOutlet weak var backingImageView: UIImageView!
    @IBOutlet weak var dimmerView: UIView!
    @IBOutlet weak var cardView: UIView!
    
    @IBOutlet weak var cardViewTopConstraint: NSLayoutConstraint!
    
    // to store backing (snapshot) image
    var backingImage: UIImage?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // update the backing image view
        backingImageView.image = backingImage
    }
}
```

<br>



As the top left and top right corner of the card view is rounded, we can round them like this : 

```swift
// ReactionViewController.swift

override func viewDidLoad() {
  super.viewDidLoad()

  // update the backing image view
  backingImageView.image = backingImage

  // round the top left and top right corner of card view
  cardView.clipsToBounds = true
  cardView.layer.cornerRadius = 10.0
  cardView.layer.maskedCorners = [.layerMinXMinYCorner, .layerMaxXMinYCorner]
}
```



Now build and run the project, tap the button, you will see the card view presented suddenly with dimmer view. Next, we want the card view to appear from the bottom smoothly and the dimmer view slowly become darker when the button is tapped.



Before proceeding to the code, let's take a look at the card view. There's two state for the card, I will name them as **.normal** and .**expanded**. Here's the top constraint value for when the card is expanded, normal and hidden (it is placed below the screen area before moving up) : 



![card state](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/cardState.png)



When user tap on the button, the card should be hidden at bottom at first, then slowly move up. Let's move the card to below the screen like this : 











