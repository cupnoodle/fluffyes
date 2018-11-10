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





