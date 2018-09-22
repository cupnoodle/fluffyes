# Intro to creating UI in code (programmatically) part 1

The debate of whether to create UI in Storyboard or Code has been so old now, there will always be developers on the different side justifying their view point, like:

> Using Storyboard can lead to complex merge conflict if there are multiple people working on the same project!

> Storyboard with lots of View Controllers takes forever to open and make Xcode freeze like a little baby 

> Making UI in code makes the UI hard to visualize, you would need to build and run the app every time to see how the UI looks like and this is time consuming.



These points are all valid, the practical answer to "**Do UI in Storyboard or code?**" is to **learn both of them**.  Some company might list ability to do UI in code in their job requirement, if you are planning to apply for a job, it is best to learn doing UI in both storyboard and code to increase your options.



In this post, we will learn how to create UILabel / UIImageView / UIButton in code and also create IBAction for UIButton. Part 2 of this post will cover how to create Auto Layout constraint in code.



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







