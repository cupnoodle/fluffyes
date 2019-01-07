# ELI 5 - Delegate

> What is a delegate? How and when do I use them?

*ELI5 = Explain Like I'm Five .*

You might have used a tableview before. To detect which row of the tableview is tapped, you will need to set the delegate in the view controller like this : 

```swift
tableView.delegate = self
```

<br>

and you add this method in the view controller : 
```swift
// a method from UITableViewDelegate
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    print("row \(indexPath.row) selected")
}
```

<br>

When you tap the second row of the tableview, you will get this console output:
![selected row](https://iosimage.s3.amazonaws.com/2018/17-delegate/row_selected.png)

<br>

>How does delegate work? Why does `tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath)` in view controller gets called automatically when I tap a row?

We will first look into the UITableView class since we are using its delegate property. As the actual UITableView class is super long and its function implementation are hidden, we will show a very simplified version (not actual) of UITableView class: 
```swift
class UITableView {
    ...
    // ALWAYS use 'weak' for delegate
    weak var delegate: UITableViewDelegate?
    
    ...
    
    // note: this code doesn't actually exist in the real UITableView header file
    // assume this function will be called when tableview receive a tap
    func tapped(at point:CGPoint){
        // detect which section the tap coordinate lands on
        let sectionInt = section(at point)
        
        // detect which row the tap coordinate lands on
        let rowInt = row(at point)
        
        // create the indexPath
        let indexPath = IndexPath(row: rowInt, section: sectionInt)
        
        // notify the delegate that a certain row is selected
        delegate?.tableView?(_ tableView: self, didSelectRowAt indexPath: indexPath) 
    }
}
```

<br>

Explanation about `UITableViewDelegate?` is located [at the bottom](#protocol) of this post.

The delegate variable is defined as **weak** to [prevent retain cycle](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID51), explaining retain cycle can be a whole post itself so we will skip it for now and leave it for another post.

Assume the `tapped(at point:CGPoint)` method will be called whenever a tap is received on the tableview. The `tapped(at point:CGPoint)` method will calculate which row is located at the tapped coordinate and notify its delegate. 

Remember in the view controller, we set the table view delegate to the view controller itself?

```swift
// ViewController.swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // set itself (this view controller) as the tableView delegate
    tableView.delegate = self
}
```

<br>

Hence we can treat the 'delegate' variable as 'ViewController' like this:
![delegate](https://iosimage.s3.amazonaws.com/2018/17-delegate/delegate.png)

So when the `delegate?.tableView?(_ tableView: self, didSelectRowAt indexPath: indexPath) ` is called, it is equivalent to calling `ViewController.tableView?(_ tableView: self, didSelectRowAt indexPath: indexPath)`. 

This is why the `tableView?(_ tableView: self, didSelectRowAt indexPath: indexPath)` method in your view controller is called when a row in the tableview is tapped.

This is the [delegation design pattern](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html#//apple_ref/doc/uid/TP40014097-CH25-ID276) you often see in iOS Development / Apple's code.

There are two question mark `?` in the `delegate?.tableView?(_ tableView: self, didSelectRowAt indexPath: indexPath)`. We will explain them one by one below.

There is a `?` in `delegate?` because the delegate can be nil. Let's say on some view controller you used a tableview, but you don't need to use any of its delegate methods like detecting which row is tapped (ie. didSelectRowAt), you can omit the delegate like this:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    tableView.dataSource = self 
    
    // tableView delegate is not set hence is nil
}

// Table view delegate methods will never be called as delegate is nil
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    print("row \(indexPath.row) selected")
}
```

<br>

This is also the reason why delegate methods (eg: `didSelectRowAt`) won't get called if you forgot to set `tableView.delegate = self`.

The second question mark `?` in 
> delegate?.tableView<strong>?</strong>(_ tableView: self, didSelectRowAt indexPath: indexPath) 

ensure that the method is only called if the delegate has implemented it. Similar to using optionals for variable, this prevents the app from crashing if the method is not implemented in the view controller. [^1]

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    tableView.dataSource = self
    tableView.delegate = self
}

// App won't crash even if you didn't implement delegate method
// as it will check if the method is implemented and only call it 
// if it is implemented.
/*
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    print("row \(indexPath.row) selected")
}
*/
```

<br>

Heck, there are [dozens of UITableView delegate methods](https://developer.apple.com/documentation/uikit/uitableviewdelegate), imagine your app crashes because you didn't implement all of them in view controllers 😱.


You might notice that all of the [UITableViewDelegate methods](https://developer.apple.com/documentation/uikit/uitableviewdelegate) always starts with `func tableView(_ tableView: UITableView` , why include the tableView object itself in the delegate method? This is to let the delegate (eg. view controller) know which tableview has called this function. 

In some case where you have two tableviews in a view controller, like this :
![two tableview in one view controller](https://iosimage.s3.amazonaws.com/2018/17-delegate/two_tableview.png)

To detect which row is tapped by the user, you have to set both tableviews' delegate to self (view controller) and implement the `didSelectAtRow` method :
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    tableView1.delegate = self
    tableView2.delegate = self
}

func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    print("row \(indexPath.row) selected")
}
```

<br>

Since both tableView1 and tableView2 calls the same delegate method, how do you know which tableView called it? This is where the `tableView` parameter in the delegate method comes in.

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    if(tableView == tableView1){
        print("user selected a row in tableView1")
    }
    
    if(tableView == tableView2){
        print(("user selected a row in tableView2")
    }
    
    print("row \(indexPath.row) selected")
    // how do I perform different action for different tableview?
}
```

<br>

Generally most delegate methods follow this pattern (passing itself as the first parameter of the method) to let user know which object initiated the delegate calls.

> When do I use delegate?

Most of the UIKit classes like UITableView, UICollectionView, UIPickerView etc has delegate methods you can use. These methods provide convenience for us as we won't need to manually write code to detect which cell is selected etc.

Some non UIKit class like [SKProductsRequest](https://developer.apple.com/documentation/storekit/skproductsrequestdelegate) also utilize delegation pattern, it will call its delegate when in-app products info is retrieved from App Store.

<br>

> When would I write a custom delegate?

Usually when I want to pass some data from a view controller back to its previous view controller, I will implement a delegate and protocol for the view controller, [like this](https://fluffy.es/3-ways-to-pass-data-between-view-controllers/#delegate). You can also write delegate methods for your own custom UIView class so that it will notify the delegate when certain action is performed.

<div class="post-subscribe" style="margin-top:1em;">
  <div class="post-subscribe-left">
    <h4> Get the demo custom delegate Xcode Project</h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/2018/17-delegate/xcode_project.png"></img>Includes explanation on how to setup your own custom delegate
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/299909614/submissions" method="post" data-drip-embedded-form="299909614">
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
                <span style="font-size: 0.8rem;">Bi-weekly ish iOS Development tips. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>

<span id="protocol"></span>
## What is protocol
**UITableViewDelegate** in  `weak var delegate: UITableViewDelegate?` is not an object type / class, it is a protocol. Meaning you can put any type of object as the delegate as long as the object's class conform to the protocol. In simplified terms, **protocol contain a list of required methods and optional methods** , if a class wants to conform to a protocol, the class have to **implement all of the required methods listed in the protocol**.

For example, a view controller can declare conformance to the UITableViewDataSource protocol by adding `UITableViewDataSource` at the end of the class declaration syntax like this: 

```swift
// Add 'UITableViewDataSource' after the class name 'UIViewController'
class ViewController: UIViewController, UITableViewDataSource {
    ...
}
```

<br>

After adding, you will see Xcode complain that the ViewController does not actually conform to UITableViewDataSource : 

![does not conform](https://iosimage.s3.amazonaws.com/2018/17-delegate/protocol_error.png)

This is because the ViewController haven't implement the required methods listed in the UITableViewDataSource protocol yet.

The UITableViewDataSource protocol looks like this :
```swift
protocol UITableViewDataSource : NSObjectProtocol {
    // protocol doesn't contain implementation of the methods
    // protocol can only contain the name of the methods
    
    
    // required methods
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell

    // optional methods will have 'optional' mentioned in front

    optional func numberOfSections(in tableView: UITableView) -> Int // Default is 1 if not implemented

    optional func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? 

    optional func tableView(_ tableView: UITableView, titleForFooterInSection section: Int) -> String?

    .... 
}
```

<br>

To make the view controller actually conform to UITableViewDataSource, we will have to implement the required methods `func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int)` and `func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)` like this :

```swift
class ViewController: UIViewController, UITableViewDataSource{

    @IBOutlet weak var tableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // tableView can assign self (view controller) as data source because 
        // self (view controller) conforms to the UITableViewDataSource 
        // as it implements the required methods below
        // and declare itself to conform UITableViewDataSource 
        // at the class declaration syntax
        tableView.dataSource = self
    }

    // Table view data source required methods    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 3
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = "ayy"
        return cell
    }

}
```

<br>

After implementing these two methods, Xcode will stop complaining as the view controller has conformed to the UITableViewDataSource protocol.

Okay, let's go back to `UITableViewDelegate` protocol. UITableViewDelegate protocol looks like this:

```swift
protocol UITableViewDelegate : NSObjectProtocol {
    
    // all methods are optional
    ....

    optional public func tableView(_ tableView: UITableView, willSelectRowAt indexPath: IndexPath) -> IndexPath?

    optional public func tableView(_ tableView: UITableView, willDeselectRowAt indexPath: IndexPath) -> IndexPath?

    optional public func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath)

    optional public func tableView(_ tableView: UITableView, didDeselectRowAt indexPath: IndexPath)

    ....
}
```

<br>

Since all methods in UITableViewDelegate are optional (no required methods to implement), all you have to do is include 'UITableViewDelegate' in the class declaration of View Controller to make it conform to UITableViewDelegate.

Like this:
```swift
// Add the 'UITableViewDelegate'
class ViewController: UIViewController, UITableViewDataSource, UITableViewDelegate {
...
}
```

<br>

Phew, that was easier than expected. Usually, Delegate protocol only contain optional methods as programmer normally won't need to use all of the method listed in the delegate protocol.


[^1]: `someObject.someMethod?()` can be used only for @objc protocols which have declared the method as optional. [Original stackoverflow post](https://stackoverflow.com/a/24167936) -


<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> Get the demo custom delegate Xcode Project</h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/2018/17-delegate/xcode_project.png"></img>Includes explanation on how to setup your own custom delegate
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/299909614/submissions" method="post" data-drip-embedded-form="299909614">
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
                <span style="font-size: 0.8rem;">Bi-weekly ish iOS Development tips. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>