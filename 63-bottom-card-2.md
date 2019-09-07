# Replicating Facebook's Draggable Bottom Card using Auto Layout - Part 2/2



In [Part 1](https://fluffy.es/facebook-draggable-bottom-card-modal-1/), we have managed to implement the show and hide card animation when user tap on button or the dimmer view. In this part, we are going to implement the card dragging animation. This post assume that you already knew about [Auto Layout](https://autolayout.fluffy.es/) and [Delegate](https://fluffy.es/eli-5-delegate/).



In this post we are going to implement this : 

![small Demo](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/smallDemo.gif)

<br>

Continuing from previous progress, so far we have implemented the function showCard() and hideCardAndGoBack() like this : 

![hide card](https://iosimage.s3.amazonaws.com/2019/62-bottom-card/showHideCard.gif)

<br>

Before we jump into implementing the card dragging animation, remember that there is two state for the card, **.normal** and **.expanded** ? 

![card state](https://iosimage.s3.amazonaws.com/2019/63-bottom-card-2/cardstate.png)



We will create an enum for the card state and a variable to store the card state, so we can know whats the current card state and perform different animation depending on the current state (eg: an expanded card cannot be further expanded).



```swift
// ReactionViewController.swift

class ReactionViewController: UIViewController {
  
  // IBOutlet here ...
  
  enum CardViewState {
      case expanded
      case normal
  }

  // default card view state is normal
  var cardViewState : CardViewState = .normal

  // to store the card view top constraint value before the dragging start
  // default is 30 pt from safe area top
  var cardPanStartingTopConstant : CGFloat = 30.0
  
  // other functions here ...
}
```

<br>



Next we will add a Pan Gesture Recognizer on the view controller's root view, to detect user's drag/pan movement and move the card view accordingly. You might be wondering "*shouldn't the pan gesture recognizer be on the card view instead of the whole view controller's view?*", if you try to drag on the area outside of card view on Facebook / Slack, the card view will move as well! 



```swift
// ReactionViewController.swift

class ReactionViewController: UIViewController {
  // ...
  override func viewDidLoad(){
    super.viewDidLoad()
  
    // ...

    // add pan gesture recognizer to the view controller's view (the whole screen)
    let viewPan = UIPanGestureRecognizer(target: self, action: #selector(viewPanned(_:)))
    
    // by default iOS will delay the touch before recording the drag/pan information
    // we want the drag gesture to be recorded down immediately, hence setting no delay
    viewPan.delaysTouchesBegan = false
    viewPan.delaysTouchesEnded = false

    self.view.addGestureRecognizer(viewPan)
  }
  
  // this function will be called when user pan/drag the view
  @IBAction func viewPanned(_ panRecognizer: UIPanGestureRecognizer) {
    // how much distance has user dragged the card view
    // positive number means user dragged downward
    // negative number means user dragged upward
    let translation = panRecognizer.translation(in: self.view)
    
    print("user has dragged \(translation.y) point vertically")
  }
}
```

<br>



Build and run the app now, and drag the card view, you will the console log as follows : 

![drag print console](https://iosimage.s3.amazonaws.com/2019/63-bottom-card-2/dragPrint.gif)



When user drag upward, the translation.y will be negative value, and when user drag downward, the translation.y will be positive value. We will add this translation.y value to the top constraint value of the card view to move it.



There's three state of panGestureRecognizer, **.began**, **.changed** and **.end** .

**.began** is right before we drag the view with finger (we just place finger on screen, haven't move yet).

**.changed** is when we are moving our finger on screen (dragging).

**.end** is when we finish drag and lift our finger away from screen.



In **.began** state, we store the current card view's top constraint constant value into the variable cardPanStartingTopConstraint : 

```swift
cardPanStartingTopConstaint = cardViewTopConstraint.constant
```

<br><br>

This is so that in the **.changed** state, we can update the card view top constraint value by adding the distance dragged (translation.y) :

```swift
self.cardViewTopConstraint.constant = self.cardPanStartingTopConstaint + translation.y
```

<br>

And we don't want the user to be able to drag the card view lesser than 30pt (< 30 pt) away from Safe Area top : 

![limit](https://iosimage.s3.amazonaws.com/2019/63-bottom-card-2/maxDrag.png)

Converting these into code :

```swift
// if the current drag distance + starting drag position is larger than 30 pt
if self.cardPanStartingTopConstaint + translation.y > 30.0 {
  // then only move the card
  self.cardViewTopConstraint.constant = self.cardPanStartingTopConstaint + translation.y
}
```

<br>



And putting them together into the **viewPanned()** function : 

```swift
@IBAction func viewPanned(_ panRecognizer: UIPanGestureRecognizer) {
  // how much has user dragged
  let translation = panRecognizer.translation(in: self.view)
  
  switch panRecognizer.state {
  case .began:
    cardPanStartingTopConstraint = cardViewTopConstraint.constant
  case .changed :
    if self.cardPanStartingTopConstraint + translation.y > 30.0 {
        self.cardViewTopConstraint.constant = self.cardPanStartingTopConstraint + translation.y
    }
  case .ended :
    print("drag ended")
    // we will do other stuff here later on
  default:
    break
  }
}
```

<br>

<br>



Now build and run the project, we should be able to drag the card view around now : 

![drag around](https://iosimage.s3.amazonaws.com/2019/63-bottom-card-2/dragAroundDemo.gif)



Looking good! But somehow something feels off, in the Facebook / Slack app, when we release the card view, it will snap to a certain position. Our current implementation doesn't have the "snap to" effect hence it feels a bit sloppy, we are going to add the "snap-to effect" next.



As the card has two state, **.normal** and **.expanded** , we want the card to snap to different state (and size) depending on the height of the card when user stop dragging.



Here's an example of what we can do : 

// snap-to effect diagram







