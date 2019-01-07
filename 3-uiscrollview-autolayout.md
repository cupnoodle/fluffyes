# How to use UIScrollView With AutoLayout in Interface Builder / Storyboard

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style>

You probably googled "How to use UIScrollView with AutoLayout" and clicked the link to this post after failed multiple attempts on making scroll view behave correctly in Interface builder. Its really quite frustrating when Xcode won't stop complaining to you that constraints are ambiguous / unsatisfiable , we will take a look on how to solve this.

At the end of this post, you will be able to design UIScrollView and its child element as below : 

![Elongated UIScrollView in IB](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/scrollViewInterfaceBuilder.png)

<div class="embed-container">
<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/-TZObMT__G8?rel=0&amp;showinfo=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>

## Step 1 : Put a scroll view into the view controller and set constraints

Assuming you have a blank view controller, drag a **Scroll View** into the view controller and set its constraint to parent (top, bottom, leading, trailling) to (0,0,0,0)  . This will pin the four side of scroll view to the parent view.

![Blank View Controller](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/blankViewController.png)
![Scroll View Element](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/scrollViewElement.png)

![Zero constraints to parent](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/zeroConstraintToParent.png)

## Step 2 : Put a view inside the scroll view and set constraints

Put a view inside the scroll view and set the view constraint against the scroll view (top, bottom, leading, trailling) to (0,0,0,0). We don't put element (eg: Label, imageview, textfield) directly inside scroll view but uses a UIView that serve as a container as it will be easier to add/remove element afterwards.

![View Element](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/viewElement.png)

![Zero constraints to parent](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/zeroConstraintToParent.png)

You might see red lines stating the layout is ambiguous, we will solve that by making two new constraint. The reason Xcode is complaining is that it doesn't know the width and height of the view, although in runtime it might render still fine, nevertheless we will still add these constraint to remove the warning.

![Ambiguous Layout](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/ambiguousLayout.png)

<kbd>Ctrl</kbd> + Drag from the view inside the scroll view to the parent view of scroll view, like this : 
![View inside scroll view to parent view](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/containerViewToParentView.png)

Then select **Equal Width**.

Next, we will explicitly **define a height** for the view inside scroll view. For demonstration purpose, we will set to 1000. You can adjust this to any number you like.
![Explicit Height](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/containerViewManualHeight.png)

Xcode should have no complain about layout issue now, yay!

## Step 3 : Elongate view controller size

Select the view controller and open the size inspector. Change the **Simulated Size** to **freeform**, then for the height type in 1000 to match the size of the view earlier.

![View controller](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/selectViewController.png)

![Freeform height](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/viewControllerSimulatedHeight.png)

Now you have an elongated view controller and can start place element inside the view of scrollview!
![Elongated View controller in IB](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/scrollViewInterfaceBuilder.png)

## Step 4 (Optional) : Cater for dynamic text in label

What if the text for the label is dynamically fetched from HTTP API? How can I have a dynamic height for the view inside of the scroll view?

First, remove the explicit height constraint for the view inside scroll view.

Then select the label and **set lines number to zero** so the label will expand based on the text. Remember to set the **leading/trailing space** of the label **to its parent** as well else Xcode doesn't know the width of the label and will keep expanding the text in 1 line (which will be outside the screen so you can't see it).
![Label line zero](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/labelLinesZero.png)

Following the guide below, set the space between the first element and the top of its container view and also the space between last element and the bottom of its container view. Vertical space between each element must be set as well so that iOS can calculate the height of the container view (the view inside the scroll view) dynamically at runtime and present it nicely inside the UIScrollView.  

![Dynamic scroll view height guide](https://iosimage.s3.amazonaws.com/2018/3-uiscrollview-autolayout/dynamicHeight.png)

The gist of dynamic height scroll view is that the vertical spacing between each element and also the parent view must be defined so that iOS layout system can calculate its height at runtime.

Happy designing in interface builder!

[Download the finished example project here](https://gitlab.com/fluffyes/fluffyscroll/repository/master/archive.zip).




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