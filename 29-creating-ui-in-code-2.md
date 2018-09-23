# Intro to creating UI in code (programmatically) part 2 - Create Auto Layout Constraint using NSLayoutAnchor

In the [previous post](https://fluffy.es/intro-to-creating-ui-in-code-1/), we have explained how to create UI elements like UILabel, UIButton etc using code. In this post, we will look into how to create Auto Layout constraint using code.



Starting from iOS9, Apple has provided us [NSLayoutAnchor](https://developer.apple.com/documentation/uikit/nslayoutanchor) which make creating constraint in code simpler.

NSLayoutAnchor are properties available on all UI elements (UIView, UILabel, UIButton etc) that can be used to set constraint on them.



Here are some Anchor types we can modify for the UI elements :

Horizontal Anchors:

1. **leadingAnchor**
2. **trailingAnchor**
3. **centerXAnchor**



Vertical Anchors:

1. **topAnchor**
2. **bottomAnchor**
3. **centerYAnchor**



Size Anchors:

1. **widthAnchor**
2. **heightAnchor**



For example, let's say we want to create a **leading**, **trailing**, **top** and **height** constraint for a UIView. In Storyboard / Interface Builder, we can create it like this : 

![create constraint](https://iosimage.s3.amazonaws.com/2018/29-creating-ui-in-code-2/constraintIB.png)



To create these constraints in code, we can use anchors like this :

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
    
    let greenView = UIView()
    greenView.backgroundColor = UIColor.green

    // Remember to set this to false, else Auto Layout won't work
    greenView.translatesAutoresizingMaskIntoConstraints = false
    
    // Remember to add the greenView to view first before creating constraint,
    // Else there will be an error when you run the app
    self.view.addSubview(greenView)
    
    greenView.leadingAnchor.constraint(equalTo: self.view.leadingAnchor, constant: 30.0).isActive = true
    // take note of the negative (-30.0), 
    // because trailing of greenView is on the left of self.view
    greenView.trailingAnchor.constraint(equalTo: self.view.trailingAnchor, constant: -30.0).isActive = true
    greenView.topAnchor.constraint(equalTo: self.view.topAnchor).isActive = true
    greenView.heightAnchor.constraint(equalToConstant: 100.0).isActive = true
}
```

<br>



We create a leading constraint for the greenView by using **greenView.leadingAnchor.constraint(equalTo: self.view.leadingAnchor, constant: 30.0).isActive = true**. This will create a leading constraint with 30.0 constant from the greenView to the root view. Remember to put `isActive = true` at the end of the constraint declaration code to activate it, else the constraint won't work.



Build and run the code, you will see the result like this : 

![Anchor result](https://iosimage.s3.amazonaws.com/2018/29-creating-ui-in-code-2/anchorResult.png)

Here is the explanation of how anchor works : 

![anchor explanation](https://iosimage.s3.amazonaws.com/2018/29-creating-ui-in-code-2/anchorExplanation.png)



There you have it! It's really simple to create Auto Layout constraint using the anchor syntax provided by Apple.



We can continue to add another label below the green view and set its center X to equal to greenView's center X : 

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
    
    let greenView = UIView()
    greenView.backgroundColor = UIColor.green
    greenView.translatesAutoresizingMaskIntoConstraints = false
    
    // Add another label
    let label = UILabel()
    label.text = "iPhone XS is a top notch phone"
    label.translatesAutoresizingMaskIntoConstraints = false
    
    // Remember to add views first before creating constraint
    self.view.addSubview(greenView)
    self.view.addSubview(label)
    
    greenView.leadingAnchor.constraint(equalTo: self.view.leadingAnchor, constant: 30.0).isActive = true
    greenView.trailingAnchor.constraint(equalTo: self.view.trailingAnchor, constant: -30.0).isActive = true
    greenView.topAnchor.constraint(equalTo: self.view.topAnchor).isActive = true
    greenView.heightAnchor.constraint(equalToConstant: 100.0).isActive = true
    
    label.centerXAnchor.constraint(equalTo: greenView.centerXAnchor).isActive = true
    
    // the top of the label is 40pt down from the bottom of greenView
    label.topAnchor.constraint(equalTo: greenView.bottomAnchor, constant: 40.0).isActive = true
}
```

<br>



Build and run the code, you will see the result like this : 

![anchor result 2](https://iosimage.s3.amazonaws.com/2018/29-creating-ui-in-code-2/anchorResult2.png)



## Safe Area Layout Guide

If we run the previous code in iPhone X, you will see that the top part of the green view is cropped 😱 :  







