# Intro to creating UI in code (programmatically) part 1 - Create UI with frame


The debate of whether to create UI in Storyboard or Code has become so old now, there will always be developers on the different side justifying their view point, like:

> Using Storyboard can lead to complex merge conflict if there are multiple people working on the same project!

> Storyboard with lots of View Controllers takes forever to open and make Xcode freeze like a little baby 

> Making UI in code makes the UI hard to visualize, you would need to build and run the app every time to see how the UI looks like and this is time consuming.



These points are all valid and there's no point to persuade the opponent as there's merit in both approach. 



The practical answer to "**Do UI in Storyboard or code?**" is to **learn both of them**.  Some company might list ability to do UI in code in their job requirement, if you are planning to apply for an iOS dev job, it is best to learn doing UI in both storyboard and code to increase your job prospects.



In this post, we will learn how to create UILabel / UIImageView / UIButton in code and also create IBAction for UIButton. Part 2 of this post will cover how to create Auto Layout constraint in code (Work in progress).


Table of contents :
1. [Create UILabel using code](#createuilabelusingcode)
2. [Create UIImageView using code](#createuiimageviewusingcode)
3. [Create UIButton and IBAction using code](#createuibuttonandibactionusingcode)
4. [Combining UILabel, UIImageView and UIButton code](#combininguilabeluiimageviewanduibuttoncode)
5. [UI Elements scoping](#uielementsscoping)
6. [Frame is relative to superview](#frameisrelativetosuperview)
7. [Extra: Creating UI in Playground](#extracreatinguiinxcodeplayground)




## Create UILabel using code

To create an UILabel in code, simply create an UILabel object in the `viewDidLoad` function of the view controller : 

```swift
// ViewController.swift

// viewDidLoad is called when the controller's root view (self.view) is loaded into memory
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.

    // Create the UILabel object with frame
    let label = UILabel()
    label.frame = CGRect(x: 50, y: 50, width: 100, height: 30)
    label.text = "Test Label"

    // Add the label to the view controller's root view
    self.view.addSubview(label)
}
```

<br>



This is the root view of the view controller : 
![root view](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/rootView.png)



Build and run the app, you should see an UILabel is added to the view. Its position and size would be like this :

![labelFrame](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/labelFrame.png)



The label's position and size is following the frame value we coded earlier, **x: 50** (50pt from left of the root view), **y: 50** (50pt from top of the root view), **width: 100** (100pt as width), **height: 30** (30pt as height).



This is equivalent of dragging a Label to the view controller in Storyboard, and setting its X, Y, Width and Height in the attribute inspector : 

![Label equivalent](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/labelEquivalent.png)



You can customize the label font and color by using the **textColor** and **font** attributes : 

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
    
    // Create the UILabel object with frame
    let label = UILabel()
    label.frame = CGRect(x: 50, y: 50, width: 100, height: 30)
    label.text = "Test Label"
    label.textColor = UIColor.red
    label.font = UIFont(name: "GillSans-Bold", size: 17.0)
    
    // Add the label to the view controller's root view
    self.view.addSubview(label)
}
```

<br>

![customizedLabel](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/customizedLabel.png)

You can check out [iOSFonts.com](http://iosfonts.com) for the list of fonts available to use in iOS. To use the system font, you can use `UIFont.systemFont(ofSize:)` . 



You can also check [Apple's documentation on UILabel](https://developer.apple.com/documentation/uikit/uilabel) for a list of attributes you can customize for UILabel. (eg: alignment, number of lines etc)



## Create UIImageView using code

Similar to UILabel, we can create an UIImageView object in the `viewDidLoad` function.




We will be using an image named 'asriel' in the Assets.xcassets folder for the image view.

![imageAsset](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/imageAsset.png)



In viewDidLoad, 

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
    
    // Create the UIImageView object with frame
    let imageview = UIImageView()
    imageView.frame = CGRect(x: 50, y: 50, width: 100, height: 100)
    imageview.image = UIImage(named: "asriel")

    // set the image to aspect fit
    imageview.contentMode = .scaleAspectFit
        
    self.view.addSubview(imageview)
}
```

<br>



Build and run the app, you should see an UIImageView is added to the view. Its position and size would be like this :

![image view](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/imageView.png)



This is equivalent of dragging an Image View to the view controller in Storyboard, setting its X, Y, Width and Height in the attribute inspector, and setting the image to 'asriel' : 

![image view equivalent](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/imageViewEquivalent.png)



You can check [Apple's documentation on UIImageView](https://developer.apple.com/documentation/uikit/uiimageview) for a list of attributes you can customize for UIImageView.



## Create UIButton and IBAction using code

You know the drill now, we are going to create an UIButton object in the `viewDidLoad` function.


```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
    
    // Create the UIButton object with frame
    let button = UIButton()
    button.frame = CGRect(x: 50, y: 50, width: 170, height: 30)

    // Set the font of the button text
    button.titleLabel?.font = UIFont.boldSystemFont(ofSize: 18.0)

    // The default text of the button
    button.setTitle("Tap Me", for: .normal)

    // The text that will appear when the button is tapped
    button.setTitle("I am being tapped", for: .highlighted)
    
    // The default color of the button text
    button.setTitleColor(UIColor.blue, for: .normal)

    // The color of the button text when the button is tapped
    button.setTitleColor(UIColor.black, for: .highlighted)
    
    self.view.addSubview(button)
}
```

<br>



The result will look like this :

![tap me gif](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/tapMe.gif)



UIButton is a bit more complex as it have a **.titleLabel** element which store the text for the button. Setting the text directly using `button.titleLabel?.text = "tap me"` somehow doesn't work, you will need to call the `setTitle(, for:)` function in order to set text for the button. Same goes to the text color as well.



To add action to the button (eg: run a function when a button is tapped), we can use the `addTarget` method :

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
    
    // Create the UIButton object with frame
    let button = UIButton()
    button.frame = CGRect(x: 50, y: 50, width: 170, height: 30)

    button.titleLabel?.font = UIFont.boldSystemFont(ofSize: 18.0)
    button.setTitle("Tap Me", for: .normal)
    button.setTitle("I am being tapped", for: .highlighted)
    
    button.setTitleColor(UIColor.blue, for: .normal)
    button.setTitleColor(UIColor.black, for: .highlighted)
  
    // Add action when user tap the button and release finger within the button area
    button.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)
    
    self.view.addSubview(button)
}

// IBAction
@IBAction func buttonTapped(_ sender: UIButton){
    print("button tapped")
}
```



We will need to manually add the function that will be executed when the button is tapped (`@IBAction func buttonTapped`), this is similar to when you control + drag the UIButton to the view controller and select action.



The **self** in the code  **.addTarget(self, action: #selector(buttonTapped(_:))** refer to the controller itself, which mean that the **ViewController.buttonTapped()** function will be executed when user tap the button and release finger within the button area.



Since [Target-Action pattern](https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/TargetAction.html) relies on Objective-C, normally you would need to add `@objc` in front of the **buttonTapped** function in order to use the **#selector()** function in Swift. If you add `@IBAction` in front of the function, Swift compiler knows that this function relies on Objective-C automatically, hence you wont need to put @objc in front of @IBAction.



## Combining UILabel, UIImageView and UIButton code

We can combine the UILabel, UIImageView and UIButton code used above like this : 

```swift
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        
        // Create the UILabel object with frame
        let label = UILabel()
        label.frame = CGRect(x: 50, y: 50, width: 100, height: 30)
        label.text = "Test Label"
        
        // Create the UIImageView object with frame
        let imageview = UIImageView()
        imageView.frame = CGRect(x: 50, y: 100, width: 100, height: 100)
        imageview.image = UIImage(named: "asriel")
        imageview.contentMode = .scaleAspectFit
        
        // Create the UIButton with frame
        let button = UIButton()
        button.frame = CGRect(x: 50, y: 220, width: 170, height: 30)
        button.titleLabel?.font = UIFont.boldSystemFont(ofSize: 18.0)
        button.setTitle("Tap Me", for: .normal)
        button.setTitle("I am being tapped", for: .highlighted)
        button.setTitleColor(UIColor.blue, for: .normal)
        button.setTitleColor(UIColor.black, for: .highlighted)
        button.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)
        
        // Add the UIelements to the view controller's root view
        self.view.addSubview(label)
        self.view.addSubview(imageview)
        self.view.addSubview(button)
    }

    // IBAction
    @objc func buttonTapped(_ sender: UIButton){
        print("button tapped")
    }
}
```

<br>



The code above will produce layout like this :  

![combined UI](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/combinedUI.png)



Pretty neat right? Now you can create UI Elements in code just like how you drag-and-drop in the Storyboard / Interface Builder. This process is similar for other UI Elements as well, like UISwitch, UISegmentedControl etc. 



You can google keywords like "UISwitch swift programmatically" if you want to know how to create UISwitch using code etc.



Since we used the **.frame** property to set the position and size of the UI element in this post, its position/size is fixed regardless of device screen size and orientation. We will explain more on how to create Auto Layout constraints for these UI element in code in part 2.



# UI Elements scoping

Most of the time you would probably want to change the text of an UILabel on another function outside of viewDidLoad. To access the UILabel on another function in the same view controller, we can move the UILabel declaration outside the viewDidLoad like this : 



```swift
class ViewController: UIViewController {
    
    // declare the label in the view controller scope, so other function can access it
    let testLabel = UILabel()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        testLabel.frame = CGRect(x: 50, y: 50, width: 80, height: 30)
        testLabel.text = "Label"
        
        // Create the UIButton with frame
        let button = UIButton()
        button.frame = CGRect(x: 50, y: 220, width: 200, height: 30)
        button.setTitle("Tap to change label", for: .normal)
        button.setTitleColor(UIColor.blue, for: .normal)
        button.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)
        
        self.view.addSubview(testLabel)
        self.view.addSubview(button)
    }

    // IBAction
    @objc func buttonTapped(_ sender: UIButton){
        // change the label text
        testLabel.text = "Wooooo"
    }
}
```

<br>





## Frame is relative to superview

One of the important thing to take note when using the **frame** property is that the **x** , **y** used in the frame is **relative to its superview**.



To demonstrate this, we will use the code below : 

```swift
override func viewDidLoad() {
super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.

    let greenView = UIView()
    greenView.frame = CGRect(x: 50, y: 50, width: 200, height: 200)
    greenView.backgroundColor = UIColor.green

    let label = UILabel()
    label.frame = CGRect(x: 50, y: 50, width: 80, height: 30)
    label.text = "test Label"
    label.backgroundColor = UIColor.gray
    // notice that label is added into the greenView
    // greenView is the parent view (superview) of the label
    greenView.addSubview(label)
    
    // root view is the parent view (superview) of the greenView
    self.view.addSubview(greenView)
}
```

 

Notice that the parent view (superview) of the label is greenView, and the parent view (superview) of greenView is the view controller's root view.



The output of the code will look like below :

![frame](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/frame.png)



Notice that the **x: 50** and **y: 50** for the **greenView** is measured **from** the top and left of the **root view**.



Whereas the **x: 50** and **y: 50** for the **label** is measured **from** the top and left of the **greenView**.



The x, y in a frame of an UI element is measured from its parent view.



## Extra : Creating UI in Xcode Playground

Since Xcode Playground doesn't have Interface Builder / Storyboard, you can only create UI using code. Now that you know how to create UI in code, you can experiment with UI in Playground! Playground let's you prototype UI quickly as you won't need to wait for the compilation of the whole project every time you run the code.



To create UI in Playground, select **Single View** when creating a new Playground.



![Single View](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/singleView.png)



Then in the window, open **Assistant Editor** and select **Live View** if it is not yet selected. Build and run the code and you should see the UI appear in the right column, awesome right? 😆



![Live View](https://iosimage.s3.amazonaws.com/2018/28-creating-ui-in-code/liveView.png)



We will explain more on how to create Auto Layout constraints for UI element in code in part 2.


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



