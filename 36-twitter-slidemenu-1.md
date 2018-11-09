# Replicating Twitter Slide Menu - Part 1



In this post, we will breakdown, analyze the Slide Menu mechanism of Twitter app, and try to replicate it using Auto Layout and Container View. This post assume you have some experience working with [Auto Layout](https://fluffy.es/making-sense-of-auto-layout/). No third party library is used in this tutorial.



Main screen of Twitter app consist of a tab view controller. If you swipe right, a side menu will appear.

![side menu and tab bar](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/sideMenuTabBar.png)



When the side menu is open, both side menu view and tab bar view appear in the same screen. The screen (which is a view controller) contain both of the side menu view controller and tab bar view controller : 

![container view](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/containerviews.png)

We can achieve this using two container view in the Main View Controller.



<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> Download the starter project to follow along the post</h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/slidePreviewSmall.png" style="max-width: 200px;"></img>Get starter Xcode project containing premade side menu and tab bar view controller.
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/947906023/submissions" method="post" data-drip-embedded-form="947906023">
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
                <input type="submit" value="Send me the project file!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips to help you become a better iOS developer.<br> No Spam. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>


If you are planning to start the project from scratch, the steps below assume you already have tab bar controller and a view controller for side menu prepared.



## Setup Container View for Tab bar controller

To accomodate the side menu and tab bar controllers, we will create a new view controller in storyboard named **MainViewController** , and drag a container view to it.



![create new file](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section3/createNewFile.png)

![mainVC](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/mainVC.png)



![Drag container view](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/dragContainerView.png)



After dragging, you will see that Xcode auto link a new view controller to the Container view. We don't need this new controller, select the link (segue) and press delete to remove the link, then delete the attached view controller.

![selectSegue](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/selectSegue.png)



Then we will link the container view to the tab bar view controller. Select the Container view, then hold control and drag it to the tab bar controller. Select "**Embed**".



![controlDrag](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/controlDragContainer.png)



![select embed](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/selectEmbed.png)



You might see the tab bar controller shrinked to match the container view size 😱, no worries, we will add some constraint to the container view so that it will match the screen size.



Let's create a leading space 0 , top space 0, equal width and equal height constraints for the container view (to its parent view) like this : 

![Constraint gif](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/multipleConstraints.gif)

Select the container view, Hold **control** and drag to its parent view. Then once the list of constraints appear, hold **shift** to select multiple constraints, then click 'Add Constraints' once you are done.



As by default Xcode align the top / bottom constraint to Safe Area, the container view will have some margin on top when viewed in iPhone X / XR / XS : 

![Safe Area](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/safeArea.png)



We want the container view to align to the screen top, not safe area, hence we will need to edit the constraint.




Select the container view and double click its Align Top constraint : 

![Double click top constraint](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/selectAlignTop.png)



Change the "**Safe Area**" in First/Second Item to "**Superview**", and then set the constant to **0** .

![Safe Area](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/changeSafeAreaToSuperView.png)



![constant zero](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/constantZeroSuperview.png)



After adjusting the top constraint, remember to set the constant of leading constraint to 0 as well.



Set the MainViewController as the initial view controller.

![initial view controller](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/initialViewController.png)



Build and run the app, you now should have a container view containing the tab bar controller 🙌. 



![super view](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/superviewArea.png)



As we will add another container view for side menu later on, we should give a label to the tab bar container view so we can identify it later. Let's label it as **ContentView**.

![label content view](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section1/contentViewLabel.png)



## Setup Container View for Side Menu View Controller

Similar to previous section, drag another container view to MainViewController, link it to the side menu view controller, setup constraints (top 0, leading 0, equal width and equal height) and change the safe area to superview.



![container view](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section2/controlDrag2.png)





Add a label to this container view, we will label it as **SideMenuView**.

![side menu label](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section2/sidemenulabel.png)



As we want the width of side menu to be 80% of screen width (so the side menu won't cover the whole screen when opened), we will change its equal width constraint to equal 80% of its parent view's width.



Select **SideMenuView** , then click on 'Edit' on its Equal width constraint, change the multipler to 0.8 . This will make the side menu view width to be 80% of its parent view.



![proportional width](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section2/multiplierWidth.png)



Build and run the app, you should see the side menu is overlapping the tab bar controller like this : 

![side menu run](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section2/sidemenurun.png)



Now we have two container view on the main view controller 🙌! 



As we are going to setup the top left profile button in the next step, let's move the side menu to the left so the tab bar controller is visible. We will set the leading constraint value of the side menu container to -350 for now.

![side menu leading](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section3/sideMenuLeading.png)




Later we will create outlet for the leading constraints of these two container view so we can manipulate their horizontal position.


## Setup top left profile button 

Notice that for every tab in the tab bar controller, they have the same top left profile button (the rounded avatar on top left) like this : 

![top left profile button](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section3/topLeftButton.png)



To reduce the repeating work of creating the top left button for each of the tab, we can create a View Controller class named '**ContentViewController**',  and then subclass view controller in each tab to it.


![create new file](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section3/createNewFile.png)



![content VC](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section3/contentVC.png)



We will add the following code to the **ContentViewController.swift** to create the left top profile button : 

```swift
class ContentViewController: UIViewController {
    
  let profileButton = UIButton(type: .custom)
  
  override func viewDidLoad() {
    super.viewDidLoad()

    // Do any additional setup after loading the view.
    profileButton.setImage(UIImage(named: "Avatar") , for: .normal)
    profileButton.contentMode = .scaleAspectFit
    profileButton.translatesAutoresizingMaskIntoConstraints = false
    
    // function performed when the button is tapped
    profileButton.addTarget(self, action: #selector(profileButtonTapped(_:)), for: .touchUpInside)
    
    // Add the profile button as the left bar button of the navigation bar
    let barbutton = UIBarButtonItem(customView: profileButton)
    self.navigationItem.leftBarButtonItem = barbutton
    
    // Set the width and height for the profile button
    NSLayoutConstraint.activate([
        profileButton.widthAnchor.constraint(equalToConstant: 35.0),
        profileButton.heightAnchor.constraint(equalToConstant: 35.0)
    ])
    
    // Make the profile button become circular
    profileButton.layer.cornerRadius = 35.0 / 2
    profileButton.clipsToBounds = true   
  }
    
  @IBAction func profileButtonTapped(_ sender: Any){

  }
}
```

<br>



And then we replace the UIViewController subclass to **ContentViewController** subclass for HomeViewController, SearchViewController, NotificationViewController and MessageViewController.


```swift
// HomeViewController.swift
// Change the UIViewController to ContentViewController
class HomeViewController: ContentViewController {
  ...
}
```

<br><br>

```swift
// SearchViewController.swift
// Change the UIViewController to ContentViewController
class SearchViewController: ContentViewController {
  ...
}
```
<br><br>

```swift
// NotificationViewController.swift
// Change the UIViewController to ContentViewController
class NotificationViewController: ContentViewController {
  ...
}
```
<br><br>

```swift
// MessageViewController.swift
// Change the UIViewController to ContentViewController
class MessageViewController: ContentViewController {
  ...
}
```

<br>



Build and run the app, we have a top left profile button for each tab now 👌!



![top left profile button](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section3/topLeftProfileButtonDone.png)





## Create outlet for constraints

To be able to modify the constraint value in code, we need to create and link outlet of constraint to the MainViewController.




Select the Side Menu Container View in storyboard, double click the leading constraint, then hold control + drag the highlighted constraint to the view controller. Name the outlet as **sideMenuViewLeadingConstraint**.



![double click constraint](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section4/doubleclickLeading.png)



![control drag to view controller](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section4/controlDragConstraint.png)



![side menu view constraint outlet](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section4/sideMenuViewConstraint.png)



Repeat the same step for the leading constraint of Tab Bar Container View (ContentView), and name the outlet as **contentViewLeadingConstraint**.



Create an outlet for the Side Menu Container View, name it as **sideMenuContainer** .

 

Your MainViewController.swift should look like this now : 

```swift
class MainViewController: UIViewController {
	
  @IBOutlet weak var sideMenuContainer: UIView!
  @IBOutlet weak var contentViewLeadingConstraint: NSLayoutConstraint!
  @IBOutlet weak var sideMenuViewLeadingConstraint: NSLayoutConstraint!

...
}
```

<br>



## Toggle side menu when top left profile button is tapped

When you tap on the top left avatar circle button on Twitter app, the side menu appears if it is not visible yet, or the side menu disappear if it is currently visible.  Now we are going to replicate this functionality.



Add a boolean variable to keep track if the side menu is visible on screen.

```swift
class MainViewController: UIViewController {
	
  @IBOutlet weak var sideMenuContainer: UIView!
  @IBOutlet weak var contentViewLeadingConstraint: NSLayoutConstraint!
  @IBOutlet weak var sideMenuViewLeadingConstraint: NSLayoutConstraint!
	
  // if side menu is visible
  var menuVisible = false
...
}
```

<br>



In viewDidLoad, we set the leading constraint of side menu container to negative width of the side menu container. So that the side menu container will be invisible at first. 



```swift
override func viewDidLoad() {
  super.viewDidLoad()

  // Do any additional setup after loading the view.
  sideMenuContainerLeadingConstraint.constant = 0 - self.sideMenuContainer.frame.size.width
}
```

<br>



![side menu leading explanation](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section4/sideMenuLeadingExplanation.png)

Next, we will add a function to toggle the side menu :  

```swift
class MainViewController: UIViewController {

    @IBOutlet weak var sideMenuContainer: UIView!
    @IBOutlet weak var contentViewLeadingConstraint: NSLayoutConstraint!
    @IBOutlet weak var sideMenuViewLeadingConstraint: NSLayoutConstraint!
    
    var menuVisible = false

    ...

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



Then in the ContentViewController, we toggle the side menu when the profile button is tapped :  

```swift
class ContentViewController: UIViewController {

  ...

  @IBAction func profileButtonTapped(_ sender: Any){
    // current view controller (self) is embed in navigation controller which is embed in tab bar controller
    // .parent means the view controller that has the container view (ie. MainViewController)
    
    if let mainVC = self.navigationController?.tabBarController?.parent as? MainViewController {
        mainVC.toggleSideMenu(fromViewController: self)
    }
  }
}
```

<br>

<br>


Build and run the app, now we have toggle-able side menu 🤘!


![toggling](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/section4/toggling.gif)



Cool right? In the next part, we will add slide gesture to show/hide menu, and segue to profile view controller when one of the table view row in side menu view controller is tapped.





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