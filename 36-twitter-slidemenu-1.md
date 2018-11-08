# Replicating Twitter-like Slide Menu - Part 1



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


## Two Container View

To accomodate the side menu and tab bar controllers, we will create a new view controller named **MainViewController** , and drag two container view to it.













