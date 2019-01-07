# How to view rounded corners, borders in Interface Builder

Are you tired of pressing compile and wait every time to see minor UI changes ?

Starting from Xcode 6, you can build custom UI and have it rendered real time in Storyboard / XIB / Interface builder.

We are going to create an UIView with rounded corners and borders in this post. First, create a UIView subclass, I name it as 'RoundedCornerView'  
![New file](https://iosimage.s3.amazonaws.com/2018/2-roundedcorners/newFile.png)
![Subclass UIView](https://iosimage.s3.amazonaws.com/2018/2-roundedcorners/subclassUIView.png)

And add the following **cornerRadius** attribute to the newly created subclass file.  

**RoundedCornerView.swift**
```swift
import UIKit

class RoundedCornerView: UIView {

	// if cornerRadius variable is set/changed, change the corner radius of the UIView
	@IBInspectable var cornerRadius: CGFloat = 0 {
		didSet {
			layer.cornerRadius = cornerRadius
			layer.masksToBounds = cornerRadius > 0
		}
	}
	
}
```

The above snippet initialize cornerRadius variable to 0 and any change on its value (didSet) will update the view layer cornerRadius and masksToBounds boolean if the cornerRadius is larger than 0.

The **@IBInspectable** keyword in front of the variable lets you set the value of cornerRadius in Interface builder!

Since @IBInspectable is based on **User-defined runtime attributes**, the variable types it support are also same as the runtime attributes : Boolean, Numbers (NSNumber, double, CGFloat etc), String, CGPoint, CGSize, CGRect, UIColor (Not CGColor), UIImage and also NSRange.  

![Supported variable type](https://iosimage.s3.amazonaws.com/2018/2-roundedcorners/typeSupported.png)


Select your desired view in interface builder, set the class to **RoundedCornerView**.
![Setting class to RoundedCornerView](https://iosimage.s3.amazonaws.com/2018/2-roundedcorners/assignRoundedCornerView.png)

Now select Attribute Inspector tab, you should see the corner radius attribute there, Xcode even auto convert your camelcased variable name to proper format!
![Attribute Inspector](https://iosimage.s3.amazonaws.com/2018/2-roundedcorners/attributeInspector.png)

You can now set the corner radius of the view in Interface Builder, but the view still looks square after setting the corner radius to 40, what did we miss? 🤔

You need to add **@IBDesignable** on top of the class definition to tell Xcode that it needs to render the view properties directly in the Interface builder.

**RoundedCornerView.swift**
```swift
import UIKit

@IBDesignable
class RoundedCornerView: UIView {
...
}
```

After adding `@IBDesignable` , open the interface builder again, you should see a rounded corner view now wew : 
![Muh rounded corners](https://iosimage.s3.amazonaws.com/2018/2-roundedcorners/muhRoundedCorner.png)  

Praise rounded corners, now you can do the same for borders : 

**RoundedCornerView.swift**
```swift
import UIKit

@IBDesignable
class RoundedCornerView: UIView {
	
	// if cornerRadius variable is set/changed, change the corner radius of the UIView
	@IBInspectable var cornerRadius: CGFloat = 0 {
		didSet {
			layer.cornerRadius = cornerRadius
			layer.masksToBounds = cornerRadius > 0
		}
	}
	
	@IBInspectable var borderWidth: CGFloat = 0 {
		didSet {
			layer.borderWidth = borderWidth
		}
	}
	
	@IBInspectable var borderColor: UIColor? {
		didSet {
			layer.borderColor = borderColor?.cgColor
		}
	}
	
}

```

<br>  

**Result** : 
![Rounded corner with borders](https://iosimage.s3.amazonaws.com/2018/2-roundedcorners/finishedView.png)

You can download the [sample Swift project here](https://git.tzhongyan.com/soulchild/roundedCorner/archive/master.zip)


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