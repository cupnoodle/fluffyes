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

  // to store the card view y position before the dragging start
  var cardPanStartingYPosition : CGFloat = 0.0
  
  // other functions here ...
}
```

<br>



