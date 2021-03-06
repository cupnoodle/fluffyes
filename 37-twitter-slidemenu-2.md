# Replicating Twitter Slide Menu - Part 2

In [Part 1](https://fluffy.es/twitter-slide-menu-1/) , we have create two container views, linked outlet to their leading constraints and implemented the toggling function. In this part, we will implement the pan gesture to show / hide menu and segue to profile view controller when user tap on the tableview row. This post assume that you already knew about [Auto Layout](https://fluffy.es/making-sense-of-auto-layout/) and [Delegate](https://fluffy.es/eli-5-delegate/).



## Create Pan Gesture to detect finger pan

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

Next, we will add some menu to the side menu's table view, and add action to segue when it is tapped.



## Setup before coding the segue

For the segue, we will push the profile view controller to the navigation controller of the selected tab.  

The code (no need to type this first, this is just an explanation of what we will do later) : 

```swift
// SideMenuViewController.swift

/* inside tableView didSelectRowAt function */

// currentActiveNav is the navigation controller of the selected tab
if let currentActiveNav = self.currentActiveNav,
let mainVC = self.parent as? MainViewController {
  // ask the MainViewController (which have the two container view) to hide the sidemenu
  // side menu view should be hidden after user tap on the menu
  mainVC.hideSideMenu()

  // instantiate the Profile View Controller from Storyboard using Identifier
  let storyboard = UIStoryboard(name: "Main", bundle: nil)
  let profileVC = storyboard.instantiateViewController(withIdentifier: "ProfileViewController")

  // push the profile view controller to the navigation controller
  currentActiveNav.pushViewController(profileVC, animated: true)
}
```

<br>



To identify which navigation controller to push the profile view controller to, we will need to create a variable to hold this value. Let's name it **currentActiveNav** :

```swift
class SideMenuViewController: UIViewController {

  var currentActiveNav : UINavigationController?
  // ...
}
```

<br>



Next, we will need to add an identifier to the profile view controller (or any other controller you want to segue to) in the Storyboard. So that we can instantiate this view controller in code like this : **storyboard.instantiateViewController(withIdentifier: "ProfileViewController")**.

![storyboard identifier](https://iosimage.s3.amazonaws.com/2018/37-twitter-slidemenu-2/storyboardIdentifier.png)



Next, we will implement the **hideSideMenu()** function in the MainViewController : 

```swift
class MainViewController: UIViewController {
  // .. below the toggleSideMenu's function
  /* its very similar to the toggleSideMenu function, 
     except it doesn't do anything when the side menu is already hidden.
     And it doesn't have the parameter fromViewController
  */
  func hideSideMenu() {
    if(menuVisible){
      UIView.animate(withDuration: 0.5, animations: {
          self.sideMenuViewLeadingConstraint.constant = 0 - self.sideMenuContainer.frame.size.width
          self.contentViewLeadingConstraint.constant = 0
          self.view.layoutIfNeeded()
      })
    
      menuVisible = !menuVisible
    }
  }
}
```

<br>



Next, we will modify the **toggleSideMenu(fromViewController:)** function in the MainViewController. We want to set the value of **currentActiveNav** to the navigation controller of the selected tab when the menu is shown.


```swift
@objc func toggleSideMenu(fromViewController: UIViewController) {
  if(menuVisible){
    UIView.animate(withDuration: 0.5, animations: {
      // hide the side menu to the left
      self.sideMenuViewLeadingConstraint.constant = 0 - self.sideMenuContainer.frame.size.width
      // move the content view (tab bar controller) to original position
      self.contentViewLeadingConstraint.constant = 0
      self.view.layoutIfNeeded()
    })
  } else {
    // set the current active navigation controller 
    // fromViewController is the view controller which called this toggleSideMenu function (view controller of the selected tab)
    self.sideMenuViewController?.currentActiveNav = fromViewController.navigationController

    self.view.layoutIfNeeded()
    UIView.animate(withDuration: 0.5, animations: {
      // move the side menu to the right to show it
      self.sideMenuViewLeadingConstraint.constant = 0
      // move the content view (tab bar controller) to the right
      self.contentViewLeadingConstraint.constant = self.sideMenuContainer.frame.size.width
      self.view.layoutIfNeeded()
    })
  }
  
  menuVisible = !menuVisible
}
```

<br>



After typing the code above, you will get an error mentioning "*Value of type 'MainViewController' has no member 'sideMenuViewController'*", this is because we haven't add the **sideMenuViewController** variable to the MainViewController yet. We will proceed to add it like this. :



```swift
class MainViewController: UIViewController {
  
  // ...
  var sideMenuViewController : SideMenuViewController?
  
  override func viewDidLoad() {
    super.viewDidLoad()

    // Do any additional setup after loading the view.
    
    // loop through all of the child view controllers (view controllers inside container views)
    // and find the side menu view controller and assign it
    for childViewController in self.childViewControllers {
      if let sideMenuVC = childViewController as? SideMenuViewController {
        sideMenuViewController = sideMenuVC
        break
      }
    }
    
    sideMenuViewLeadingConstraint.constant = 0 - self.sideMenuContainer.frame.size.width
  }
  
  // ...
}
```

<br>

We have finished preparing for the setup, next, we will setup the table view and implement the tap action in the side menu.


## Setup Table view and row tap action

Let's create an outlet for the table view in the side menu view controller (hold control and drag), and name it **menuTableView**.



![create outlet](https://iosimage.s3.amazonaws.com/2018/37-twitter-slidemenu-2/menuOutlet.png)



![create outlet 2](https://iosimage.s3.amazonaws.com/2018/37-twitter-slidemenu-2/menuOutlet2.png)



Also create a variable **cellIdentifier** to hold the identifier for cell reuse : 

```swift
class SideMenuViewController: UIViewController {
    
    @IBOutlet weak var menuTableView: UITableView!
    
    let cellIdentifier = "cellIdentifier"
    var currentActiveNav : UINavigationController?
  
//...
}
```

<br>



In viewDidLoad, set the table view datasource/ delegate to self : 

```swift
// SideMenuViewController.swift

override func viewDidLoad() {
    super.viewDidLoad()
    menuTableView.register(UITableViewCell.self, forCellReuseIdentifier: cellIdentifier)
    menuTableView.dataSource = self
    menuTableView.delegate = self
}
```

<br>



Then implement the datasource / delegate method : 

```swift
// SideMenuViewController.swift

class SideMenuViewController: UIViewController {
// ...
}

extension SideMenuViewController : UITableViewDataSource {
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
      return 3
  }
  
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: cellIdentifier, for: indexPath)
    
    switch indexPath.row {
    case 0:
      cell.textLabel?.text = "Profile"
    case 1:
      cell.textLabel?.text = "List"
    case 2:
      cell.textLabel?.text = "Moments"
    default:
      cell.textLabel?.text = ""
    }
    
    return cell
  }
}

extension SideMenuViewController : UITableViewDelegate {
  func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    tableView.deselectRow(at: indexPath, animated: true)
    
    if let currentActiveNav = self.currentActiveNav,
      let mainVC = self.parent as? MainViewController {
      mainVC.hideSideMenu()
      let storyboard = UIStoryboard(name: "Main", bundle: nil)
      let profileVC = storyboard.instantiateViewController(withIdentifier: "ProfileViewController")
      currentActiveNav.pushViewController(profileVC, animated: true)
    }
  }
}
```

<br>



Build and run the app, open the side menu and tap on any row, you should see this : 

![push segue](https://iosimage.s3.amazonaws.com/2018/37-twitter-slidemenu-2/pushSegue.gif)



You have now implemented Twitter slide menu without using any library / Cocoapods 🎉!  Container views helped us a lot on implementing the slide menu, now you can use one less library when you want to implement slide menu 🙌.



<br>

<script src="https://gumroad.com/js/gumroad.js"></script>
<p style="text-align:center;">
<br>
Want to dive in the completed project and try it out?<br><br>
<a class="gumroad-button" href="https://gum.co/twitterslidemenu" target="_blank">Get the completed project</a>
</p>


<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> Learn and understand how Auto Layout works</h4>
    <span style="font-size:0.8em;"> 
    Can't wrap your head around Auto Layout? <br><br>
    In this free 6 lesson email course, you will learn :
    <ol>
        <li>How Auto Layout determines the position and size of a view 📏</li>
        <li>How to solve red lines (missing / conflicting constraint) in Interface Builder❗️</li>
        <li>How to create dynamic height label and using it for dynamic layout⚡️</li>
    </ol>
    </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/448009646/submissions" method="post" data-drip-embedded-form="448009646">
                <div style="margin-bottom: 0.5rem;">
                    <label for="drip-firstname">Name<span style="color:#952B45;">*</span></label><br />
                    <input type="text" id="drip-firstname" name="fields[firstname]" value="" />
                </div>
                <div>
                    <label for="drip-email">Email Address<span style="color:#952B45;">*</span></label><br />
                    <input type="email" id="drip-email" name="fields[email]" value="" />
                </div>
              <div>
                <br>
                <input type="submit" value="Send me Lesson 1" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips to help you become a better iOS developer.<br> No Spam. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>


