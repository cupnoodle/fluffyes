# Setting background color on a Stack View

In a hurry and just want jump to the code? [Click here](#tldr)



You might have tried to set background color on a stack view before, be it using Interface builder or using code like `stackView.backgroundColor = UIColor.green` , but it doesn't work as setting background color have no effect on Stack view. 



If you type in **UIStackView** in your code and <kbd>Command</kbd> + click it, you can see that its header file has comments mentioning that it is a **non-rendering** subclass of UIView, meaning the stack view will never be drawn (render) on screen, hence setting background color has no effect.



![stack view header](https://iosimage.s3.amazonaws.com/2019/52-stack-view-background-color/stackViewHeader.png)





## Arranged Subviews

Stack view works by having an array named **arrangedSubviews** , when we add views into this array, stack view will manage the layout for these views according to the axis, alignment and distribution set for the stack view. When we drag and drop a view into stack view in the storyboard, we are actually putting the view into the stack view's **arrangedSubviews**.



![storyboard arranged subviews](https://iosimage.s3.amazonaws.com/2019/52-stack-view-background-color/storyboardStackview.png)



You might have forgotten that Stackview still has the **subviews** array property like all other UIView subclasses have. When we add views into **arrangeSubviews**, stack view will also add it into its **subviews** automatically. One thing to take note is that Stack view will auto arrange and layout its arrangedSubviews, but not its subviews.



![subviews](https://iosimage.s3.amazonaws.com/2019/52-stack-view-background-color/stackviewSubviews.png)



As stack view won't be rendered on screen, one way to add background color to it is to add a view into the stack view's **subviews** directly and set background color for that view.



## Solution

For the **subviews** array, the view furthest in the back (background) has the index 0, and the top most view in hierarchy has the largest index. ie. subviews[0] is the background view. 



To set a background color on the stack view, we can add an UIView to the stack view's **subviews** (with index 0 , to serve as background so it won't block other views in arrangedSubviews), set its background color, and then pin its edge to the stack view. (top, bottom, leading and trailing)

<span id="tldr"></span>

```swift
let backgroundView = UIView()
backgroundView.backgroundColor = UIColor.gray

backgroundView.translatesAutoresizingMaskIntoConstraints = false

// put background view as the most background subviews of stack view
stackView.insertSubview(backgroundView, at: 0)

// pin the background view edge to the stack view edge
NSLayoutConstraint.activate([
    backgroundView.leadingAnchor.constraint(equalTo: stackView.leadingAnchor),
    backgroundView.trailingAnchor.constraint(equalTo: stackView.trailingAnchor),
    backgroundView.topAnchor.constraint(equalTo: stackView.topAnchor),
    backgroundView.bottomAnchor.constraint(equalTo: stackView.bottomAnchor)
])
```

<br>



Here's the resulting output : 

![stack view background color](https://iosimage.s3.amazonaws.com/2019/52-stack-view-background-color/stackviewBG.png)



Currently there's no way to do this in Storyboard as dropping a view inside stackview in Storyboard will automatically place it in **arrangedSubviews** instead of subviews.



