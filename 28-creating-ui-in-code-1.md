# Intro to creating UI in code (programmatically) part 1

The debate of whether to create UI in Storyboard or Code has been so old now, there will always be developers on the different side justifying their view point, like:

> Using Storyboard can lead to complex merge conflict if there are multiple people working on the same project!

> Storyboard with lots of View Controllers takes forever to open and make Xcode freeze like a little baby 

> Making UI in code makes the UI hard to visualize, you would need to build and run the app every time to see how the UI looks like and this is time consuming.



These points are all valid, the practical answer to "**Do UI in Storyboard or code?**" is to **learn both of them**.  Some company might list ability to do UI in code in their job requirement, if you are planning to apply for a job, it is best to learn doing UI in both storyboard and code to increase your options.



In this post, we will learn how to create UILabel / UIImageView / UIButton in code and also create IBAction for UIButton. Part 2 of this post will cover how to create Auto Layout constraint in code.



## Create UILabel using code

To create an UILabel in code, simply create an UILabel object in the `viewDidLoad` function of the view controller : 

```swift
// ViewController.swift

// viewDidLoad is called when the controller's root view (self.view) is loaded into memory
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.

    // Create the UILabel object with frame
    let label = UILabel(frame: CGRect(x: 50, y: 50, width: 100, height: 30))
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
    let label = UILabel(frame: CGRect(x: 50, y: 50, width: 100, height: 30))
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
    let imageview = UIImageView(frame: CGRect(x: 50, y: 50, width: 100, height: 100))
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
    let button = UIButton(frame: CGRect(x: 50, y: 50, width: 170, height: 30))

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
    let button = UIButton(frame: CGRect(x: 50, y: 50, width: 170, height: 30))

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
        let label = UILabel(frame: CGRect(x: 50, y: 50, width: 100, height: 30))
        label.text = "Test Label"
        
        // Create the UIImageView object with frame
        let imageview = UIImageView(frame: CGRect(x: 50, y: 100, width: 100, height: 100))
        imageview.image = UIImage(named: "asriel")
        imageview.contentMode = .scaleAspectFit
        
        // Create the UIButton with frame
        let button = UIButton(frame: CGRect(x: 50, y: 220, width: 170, height: 30))
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



Pretty neat right? Now you can create UI Elements in code just like how you drag-and-drop in the Storyboard / Interface Builder. 



Since we used the **.frame** property to set the position and size of the UI element in this post, its position/size is fixed regardless of device screen size and orientation. We will explain more on how to create Auto Layout constraints for these UI element in code for the next part.



## Extra : Frame is relative to superview

One of the important thing to take note when using the **frame** property is that the **x** , **y** used in the frame is **relative to its superview**.



To demonstrate this, we will use the code below : 

```swift
override func viewDidLoad() {
super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.

    let greenView = UIView(frame: CGRect(x: 50, y: 50, width: 200, height: 200))
    greenView.backgroundColor = UIColor.green

    let label = UILabel(frame: CGRect(x: 50, y: 50, width: 80, height: 30))
    label.text = "test Label"
    label.backgroundColor = UIColor.gray
    // notice that label is added into the greenView
    // greenView is the parent view (superview) of the label
    greenView.addSubview(label)
    
    // root view is the parent view (superview) of the greenView
    self.view.addSubview(greenView)
}
```

 