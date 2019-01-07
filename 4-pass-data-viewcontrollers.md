# 3 ways to pass data between view controllers (forth and back)

So now you have set up different view controllers in the storyboard and wanting to pass the input data gotten from one view controller to another for display purpose or processing. This post will show you 3 ways to do it and discuss when to use them.

The 3 ways are :
1. [Segue, in **prepareForSegue** method (Forward)](#segue)
2. [Delegate (Backward)](#delegate)
3. [Setting variable directly (Forward)](#direct)


[When to use them](#whentouse)

<span id="segue"></span>

## 1 - Segue, in prepareForSegue method

Assume you have two view controller set up in storyboard like this, 
![Input View Controller and Display View Controller](https://iosimage.s3.amazonaws.com/2018/4-pass-data-viewcontrollers/inputVC_displayVC.png)

Link the InputViewController to DisplayViewController by clicking the yellow round icon on top of InputViewController, then press and hold <kbd>control</kbd> and drag it to DisplayViewController. After releasing the mouse click, select '**Show**' inside Manual Segue.

![Make a segue between view controllers](https://iosimage.s3.amazonaws.com/2018/4-pass-data-viewcontrollers/makeSegue.png)

Then select the segue between view controller, and set an identifier for it, we will use this identifier on the next step. Usually my naming convention will be "sourceVCtoDestinationVC".

![Set Identifier for Segue](https://iosimage.s3.amazonaws.com/2018/4-pass-data-viewcontrollers/setSegueIdentifier.png)

When the **Next** button is tapped, we will tell the InputViewController to perform the segue. The segue identifier is used here. This method tells the InputViewController to perform a segue to DisplayViewController. 

The **sender** parameter refer to the object which initiated this segue, we put 'self' to indicate that this segue is initiated by the InputViewController.

```swift
class InputViewController: UIViewController {

    ...
    
	@IBAction func nextTapped(_ sender: Any) {
		self.performSegue(withIdentifier: "InputVCToDisplayVC", sender: self)
	}
    
    ...
}
```

<br>
<br>  

You can now build the app and tap **Next**, it will show the display view controller, but the data is not passed yet.  

In InputViewController, if there is a `MARK: - Navigation`   , uncomment the code below it. Else add the following code :  
```swift
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    }
```
<br>
<br>

The prepare(for segue:, sender:) method will be called by the view controller just before a segue is performed. You can pass the data to the next view controller here.

Lets put a `name` string variable in Display View Controller to hold the name input data.
```swift
class DisplayViewController: UIViewController {
    ...
	var name: String?
    ...
}
```
<br>
<br>

In the `prepareForSegue` method in InputViewController, add the following  code :  
```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
	if(segue.identifier == "InputVCToDisplayVC"){
			let displayVC = segue.destination as! DisplayViewController
			displayVC.name = nameTextField.text
	}
}
```
<br>
<br>

This will set the **name** variable of the DisplayViewController before it is displayed. Since we already know the destination of the segue (the view controller to be presented) is a DisplayViewController, we can safely downcast it to DisplayViewController.  

Inside the DisplayViewController **viewDidLoad** method, you can set `nameLabel.text = name` . Now you have successfully pass data forward from one controller to another using `prepareForSegue` method.

<span id="delegate"></span>
## 2 - Delegate
So now you want to pass a data to the previous view controller (InputViewController) as the current view controller (DisplayViewController) will be dismissed/popped. How do we do this as segue is uni directional? Assume the view controller is presented using the segue way shown above , we will add a protocol and a weak variable named 'delegate' to the destination (DisplayViewController) view controller.

```swift
// DisplayViewController.swift

protocol DisplayViewControllerDelegate : NSObjectProtocol{
	func doSomethingWith(data: String)
}

class DisplayViewController: UIViewController {
	
	weak var delegate : DisplayViewControllerDelegate?
}
```
<br>
<br>

This is the [Delegate Concept](https://fluffy.es/eli-5-delegate/) and [Protocol](https://developer.apple.com/library/content/documentation/General/Conceptual/DevPedia-CocoaCore/Protocol.html#//apple_ref/doc/uid/TP40008195-CH45-SW1) used by Apple Cocoa Library. This is how the `tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath)` work when you tap on a row in table view. The reason for using **weak var** for the delegate is to avoid retain cycle (both source view controller and destination view controller has a strong reference to each other, causing ARC to not free the memory), [read more about ARC memory management here](https://www.raywenderlich.com/134411/arc-memory-management-swift) .

The gist of it is that you don't need to know what class is the source controller, as long as the source controller implements the method **doSomethingWith(data:)**, it will work. There might be cases where multiple controller has their own segue to the same destination view controller, it will be cumbersome to remember what class is the source view controller, so we will just make sure all of the source view controllers conform to the **DisplayViewControllerDelegate** protocol by implementing the **doSomethingWith(data:)** method.

In the **prepareForSegue** method of the source view controller, set the **delegate** variable to self.

```swift
// InputViewController.swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
	if(segue.identifier == "InputVCToDisplayVC"){
		let displayVC = segue.destination as! DisplayViewController
		displayVC.delegate = self
	}
}
```
<br>
<br>

You might see Xcode complain that `Cannot assign value of type 'InputViewController' to type 'DisplayViewControllerDelegate?'` , this is because you didn't conform the view controller to **DisplayViewControllerDelegate**, you can fix it by adding it to the back end view controller declaration line :  
```swift
class InputViewController: UIViewController, DisplayViewControllerDelegate {
...
}
```
<br>
<br>

Shortly after, you will see Xcode complains `Type 'InputViewController' does not conform to protocol 'DisplayViewControllerDelegate'` , this is because you haven't implement the **doSomethingWith(data:)** declared in the **DisplayViewControllerDelegate** protocol. Simply add the method to the view controller to fix it :  

```swift
// InputViewController.swift
func doSomethingWith(data: String) {
// Do something here after receiving data from destination view controller
}
```
<br>
<br>

So now when you want to pass a data from destination view controller back to the source view controller, you can call the delegate like this : 
```swift
// DisplayViewController.swift
// Whenever you want to pass a data back
if let delegate = delegate{
	delegate.doSomethingWith(data: "John Cena")
}
```
<br>
<br>

Then the **doSomethingWith(data:)** method on the source view controller will be called.


<span id="direct"></span>
## 3 - Setting variable directly
This step is for presenting view controller or pushing view controller programmatically without using storyboard segue.

In some cases, a view controller might have multiple entry point from other view controllers, which is not following a specified flow. You can use **Storyboard ID** to initialize the destination view controller programmatically. Select the view controller in storyboard, and set the "Storyboard ID" in the Identity Inspector tab.

![Set the Storyboard ID](https://iosimage.s3.amazonaws.com/2018/4-pass-data-viewcontrollers/setStoryboardID.png)

Then in the source view controller, you can present it like this : 
```swift
// InputViewController.swift
@IBAction func nextTapped(_ sender: Any) {
	// the name for UIStoryboard is the file name of the storyboard without the .storyboard extension
	let displayVC : DisplayViewController = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "DisplayVC") as! DisplayViewController
	displayVC.name = "John Cena"
		
	self.present(displayVC, animated: true, completion: nil)
}
```
<br>
<br>

If you are using xib, refer to this [Stack Overflow answer](https://stackoverflow.com/questions/37046573/loading-viewcontroller-from-xib-file).

<span id="whentouse"></span>

## When to use which method for passing data

### For forward data passing
If you are using segue to go to next view controller, use the **prepareForSegue**. The pros of this method is that this is straight forward and easy, the cons is that this method is only effective for simple segue flow.

If you are using view controller with Xib, without segue flow, or programmatically, [instantiate the view controller programmatically and set variable directly (Forward)](#direct).

### For backward data passing
Use the [delegate pattern](#delegate).

### Notes
Avoid using NSNotification to pass data as this will make your life hard when debugging later (you need to check which view controllers has the observer for a certain NSNotification Name).

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