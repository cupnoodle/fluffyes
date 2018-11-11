# Replicating Twitter Slide Menu - Part 2

In [Part 1](https://fluffy.es/twitter-slide-menu-1/) , we have create two container views, linked outlet to their leading constraints and implemented the toggling function. In this part, we will implement the pan gesture to show / hide menu and segue to profile view controller when user tap on the tableview row. This post assume that you already knew about [Auto Layout](https://fluffy.es/making-sense-of-auto-layout/).



# Create Pan Gesture to detect finger pan

As the side menu will move according to the distance we swiped in the app, we will create a Pan Gesture recognizer to capture the movement of finger on the ContentViewController (which every view controllers in tab bar are subclassed from, so the pan gesture will apply on each of them).



```swift
class ContentViewController: UIViewController {
    
  override func viewDidLoad() {
    super.viewDidLoad()

    // Do any additional setup after loading the view.
    // profile button stuff here...
    
    // gesture recognizer, to detect the gesture and perform action
    let panGestureRecognizser = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)) )
    view.addGestureRecognizer(panGestureRecognizser)
  }

  // function to handle the pan gesture
  @objc func handlePan(_ recognizer: UIPanGestureRecognizer){
    // how much distance have user finger moved since touch start (in X and Y)
    let translation = recognizer.translation(in: self.view)
    
    // for demonstration purpose below, you can ignore this line
    print("panned x: \(translation.x), y: \(translation.y)")

    // the main view controller that have two container view
    guard let mainVC = self.navigationController?.tabBarController?.parent as? MainViewController else {
      return
    }    
  }
}
```

<br>



Here's the output of the translation when panned : 

![pan output](https://iosimage.s3.amazonaws.com/2018/37-twitter-slidemenu-2/panSample.gif)



When user's finger move right, the translation's X value is positive; when user's finger move left, the  translation's X value is negative.



We will add some code to the **handlePan** function to modify the leading constraint values of Side Menu Container View and Tab bar Container view : 

```swift
// function to handle the pan gesture
@objc func handlePan(_ recognizer: UIPanGestureRecognizer){
  let translation = recognizer.translation(in: self.view)
  
  // the main view controller that have two container view
  guard let mainVC = self.navigationController?.tabBarController?.parent as? MainViewController else {
    return
  }
  
  // if side menu is not visisble
  // and user finger move to right
  // and the distance moved is smaller than the side menu's width
  if(!mainVC.menuVisible && translation.x > 0.0 && translation.x <= mainVC.sideMenuContainer.frame.size.width) {
    // move the side menu to the right
    mainVC.sideMenuViewLeadingConstraint.constant = 0 - mainVC.sideMenuContainer.frame.size.width + translation.x
    
    // move the tab bar controller to the right
    mainVC.contentViewLeadingConstraint.constant = 0 + translation.x
  }
  
  // if the side menu is visible
  // and user finger move to left
  // and the distance moved is smaller than the side menu's width
  if(mainVC.menuVisible && translation.x < 0.0 && translation.x >= 0 - mainVC.sideMenuContainer.frame.size.width) {
    // move the side menu to the left
    mainVC.sideMenuViewLeadingConstraint.constant = 0 + translation.x
    
    // move the tab bar controller to the left
    mainVC.contentViewLeadingConstraint.constant = mainVC.sideMenuContainer.frame.size.width + translation.x
  }
}
```

<br>



Build and run the app, and try dragging to the right : 

![Pan stuck](https://iosimage.s3.amazonaws.com/2018/37-twitter-slidemenu-2/panStuck.gif)


When we release the drag / finger , the side menu is stuck on the last position we dragged! 😬 . When you release finger in the middle of drag on the Twitter app, it should continue to move until the menu is fully opened / closed. Next, we will add some code for this 'snap-to-nearest' feature.



Modify the **handlePan** function to include the 'snap-to-nearest' feature : 

```swift
// function to handle the pan gesture
@objc func handlePan(_ recognizer: UIPanGestureRecognizer){
  let translation = recognizer.translation(in: self.view)
  
  // the main view controller that have two container view
  guard let mainVC = self.navigationController?.tabBarController?.parent as? MainViewController else {
    return
  }
  
  // when user lift up finger / end drag
  if(recognizer.state == .ended || recognizer.state == .failed || recognizer.state == .cancelled){
      
    if(mainVC.menuVisible){
      // user finger moved to left before ending drag
      if(translation.x < 0){
        // toggle side menu (to fully hide it)
        mainVC.toggleSideMenu(fromViewController: self)
      }
    } else {
      // user finger moved to right and more than 100pt
      if(translation.x > 100.0){
        // toggle side menu (to fully show it)
        mainVC.toggleSideMenu(fromViewController: self)
      } else {
        // user finger moved to right but too less
        // hide back the side menu (with animation)
        mainVC.view.layoutIfNeeded()
        UIView.animate(withDuration: 0.5, animations: {
          mainVC.sideMenuViewLeadingConstraint.constant = 0 - mainVC.sideMenuContainer.frame.size.width
          mainVC.contentViewLeadingConstraint.constant = 0
          mainVC.view.layoutIfNeeded()
        })
      }
    }
    
    // early return so code below won't get executed
    return
  }
        
    
  if(!mainVC.menuVisible && translation.x > 0.0 && translation.x <= mainVC.sideMenuContainer.frame.size.width) {
    mainVC.sideMenuViewLeadingConstraint.constant = 0 - mainVC.sideMenuContainer.frame.size.width + translation.x
    
    mainVC.contentViewLeadingConstraint.constant = 0 + translation.x
  }
    
  if(mainVC.menuVisible && translation.x >= 0 - mainVC.sideMenuContainer.frame.size.width && translation.x < 0.0) {
    mainVC.sideMenuViewLeadingConstraint.constant = 0 + translation.x
    
    mainVC.contentViewLeadingConstraint.constant = mainVC.sideMenuContainer.frame.size.width + translation.x
  }
}
```

<br>



Build and run the app, and try to release while dragging. The side menu should snap to fully open / close position now 🤘.

![panSnap](https://iosimage.s3.amazonaws.com/2018/37-twitter-slidemenu-2/panSnap.gif)



We have successfully added the swipe to open/close gesture for the side menu, Awesome!

