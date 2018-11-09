# Replicating Twitter Slide Menu - Part 1



In this post, we will breakdown, analyze the Slide Menu mechanism of Twitter app, and try to replicate it using Auto Layout and Container View. This post assume you have some experience working with [Auto Layout](https://fluffy.es/making-sense-of-auto-layout/).



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

To accomodate the side menu and tab bar controllers, we will create a new view controller named **MainViewController** , and drag a container view to it.



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









