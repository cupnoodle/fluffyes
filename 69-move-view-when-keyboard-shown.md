# Move view when keyboard is shown



Most likely you have worked on an app with multiple textfields, and when the keyboard show up, there's a chance that your text field will get covered by the keyboard! One solution for this is to move the view up when a keyboard is presented.



![keyboard](https://iosimage.s3.amazonaws.com/2019/69-move-view-when-keyboard-shown/keyboard.png)



Right before a keyboard is shown on the screen, iOS will send a notification named **UIResponder.keyboardWillShowNotification** to your current active UIResponder (can be your view or view controller). The notification contains information of the keyboard such as the size of they keyboard, which you can use to calculate how much distance should the view move upward.



We can get the frame of the keyboard by accessing the key **UIResponder.keyboardFrameEndUserInfoKey** from the userInfo dictionary of the notification. I noticed some of the StackOverflow answers use the key *keyboardFrameBeginUserInfoKey* instead of keyboardFrameEndUserInfoKey , which might produce incorrect output as keyboardFrameBeginUserInfoKey contains the frame info of the keyboard **before** the keyboard animation starts (animation of keyboard move up from bottom of the screen). 



[Apple documentation](https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/KeyboardManagement/KeyboardManagement.html) recommends using keyboardFrameEndUserInfoKey as this contain the frame of the keyboard **after** the animation has ended.



```swift
// the frame of the keyboard
let keyboardSize = (notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue)?.cgRectValue
```

<br>



We can then use this keyboardSize's height to move the root view Y origin upward like this : 

```swift
// ViewController.swift

// in ViewDidLoad, observe the keyboardWillShow notification
override func viewDidLoad() {
    super.viewDidLoad()
  
    // call the 'keyboardWillShow' function when the view controller receive notification that keyboard is going to be shown
    NotificationCenter.default.addObserver(self, selector: #selector(ViewController.keyboardWillShow), name: UIResponder.keyboardWillShowNotification, object: nil)
}

@objc func keyboardWillShow(notification: NSNotification) {       
    guard let keyboardSize = (notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue)?.cgRectValue else {
       // if keyboard size is not available for some reason, dont do anything
       return
    }
  
  // move the root view up by the distance of keyboard height
  self.view.frame.origin.y = 0 - keyboardSize.height
}
```

<br>



And when user dismiss the keyboard, another notification will be fired, **UIResponder.keyboardWillHideNotification**. We then observe for this notification and move back the root view frame origin to **0** (it's original origin is zero).



```swift
// ViewController.swift

// in ViewDidLoad, observe the keyboardWillShow notification
override func viewDidLoad() {
    super.viewDidLoad()
  
    // call the 'keyboardWillShow' function when the view controller receive the notification that a keyboard is going to be shown
    NotificationCenter.default.addObserver(self, selector: #selector(ViewController.keyboardWillShow), name: UIResponder.keyboardWillShowNotification, object: nil)
  
      // call the 'keyboardWillHide' function when the view controlelr receive notification that keyboard is going to be hidden
    NotificationCenter.default.addObserver(self, selector: #selector(ViewController.keyboardWillHide), name: UIResponder.keyboardWillHideNotification, object: nil)
}

@objc func keyboardWillShow(notification: NSNotification) {
        
    guard let keyboardSize = (notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue)?.cgRectValue else {
       // if keyboard size is not available for some reason, dont do anything
       return
    }
  
  // move the root view up by the distance of keyboard height
  self.view.frame.origin.y = 0 - keyboardSize.height
}

@objc func keyboardWillHide(notification: NSNotification) {
  // move back the root view origin to zero
  self.view.frame.origin.y = 0
}
```

<br>



This will produce the output like below : 

![diagram of changing view origin](https://iosimage.s3.amazonaws.com/2019/69-move-view-when-keyboard-shown/vieworigin.png)



If you just want to move up the view origin y when keyboard is shown, this should do the trick. However this simple approach might push some textfield too high of the screen :

![textfield too high](https://iosimage.s3.amazonaws.com/2019/69-move-view-when-keyboard-shown/toohigh.gif)





In this scenario, the username field is too high far off the screen and user can't view what they are typing! 😅



One of the solution for this scenario is to check if the text field will be covered by the keyboard when the keyboard appear, if it will be covered , then move the whole view up; if not, then just leave the view as it is and show the keyboard since the text field is still visible.



![should move](https://iosimage.s3.amazonaws.com/2019/69-move-view-when-keyboard-shown/shouldmove.png)



How can we check if a textfield will get covered by the keyboard? First we get the **bottomY value of the textfield** (the bottom edge of the textfield), and compare it to the **visible range** (the screen area that is still visible after keyboard is presented). If the bottomY value is larger than the visible range (ie. the bottomY value of textfield is larger than the bottomY value of visible range), then only we move the view up.



![check is in visible range](https://iosimage.s3.amazonaws.com/2019/69-move-view-when-keyboard-shown/checkframe.png)





As there are multiple textfields on the screen, how do we get the bottomY of the **current selected textfield** by the user? 



We can declare a variable to keep track of the current active textfield, say **activeTextField**, and assign the selected textfield to this variable when user select a textfield.



We set the textfield delegate to self (to the view controller), and we listen to when user select a textfield in the UITextFieldDelegate function, **textFieldDidBeginEditing**.

```swift
// ViewController.swift
class ViewController: UIViewController {

  @IBOutlet weak var usernameTextField: UITextField!

  @IBOutlet weak var passwordTextField: UITextField!

  // ... other text field

  // to store the current active textfield
  var activeTextField : UITextField? = nil
  
  override func viewDidLoad() {
    super.viewDidLoad()
    
    // ....
    
    // add delegate to all textfields to self (this view controller)
    usernameTextField.delegate = self
    passwordTextField.delegate = self
    emailTextField.delegate = self
  }
}

extension ViewController : UITextFieldDelegate {
  // when user select a textfield, this method will be called
  func textFieldDidBeginEditing(_ textField: UITextField) {
    // set the activeTextField to the selected textfield
    self.activeTextField = textField
  }
	
  // when user click 'done' or dismiss the keyboard
  func textFieldDidEndEditing(_ textField: UITextField) {
    self.activeTextField = nil
  }
}

```

<br>



And then we compare the activeTextField's maxY (bottomY) value to the visibleRange in the **keyboardWillShow** function : 

```swift
@objc func keyboardWillShow(notification: NSNotification) {

  guard let keyboardSize = (notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue)?.cgRectValue else {

    // if keyboard size is not available for some reason, dont do anything
    return
  }

  var shouldMoveViewUp = false

  // if active text field is not nil
  if let activeTextField = activeTextField {

    let bottomOfTextField = activeTextField.convert(activeTextField.bounds, to: self.view).maxY;
    
    let topOfKeyboard = self.view.frame.height - keyboardSize.height

    // if the bottom of Textfield is below the top of keyboard, move up
    if bottomOfTextField > topOfKeyboard {
      shouldMoveViewUp = true
    }
  }

  if(shouldMoveViewUp) {
    self.view.frame.origin.y = 0 - keyboardSize.height
  }
}
```

<br>



Build and run your app, you should see the view only move up if the text field selected is located near the keyboard : 

![move up if needed](https://iosimage.s3.amazonaws.com/2019/69-move-view-when-keyboard-shown/moveifneeded.gif)





If your view have more than 3 textfields, I recommend putting them inside a scrollview ([You can check out how to place UI inside scrollview in Storyboard here](https://fluffy.es/scrollview-storyboard-xcode-11/)) and use some code to scroll up the scrollview when a keyboard is shown, we will explain more about this in the next section.



## Scrolling up scrollview when keyboard is shown

Assuming you have set up a scrollview with textfields inside it like this : 

![scrollview](https://iosimage.s3.amazonaws.com/2019/69-move-view-when-keyboard-shown/scrollView.png)



We can add bottom padding into the scrollview using its **contentInset** property, so the content inside will be moved up when a keyboard is shown.



You can think of contentInset as an additional padding extending from the content area edges.



Here's an illustration on how contentInset works, and how to use it to adapt for the keyboard.



![content Insets](https://iosimage.s3.amazonaws.com/2019/69-move-view-when-keyboard-shown/contentInset.png)





When the keyboard will show / will hide notification is observed, we can modify the contentInsets of the scrollview : 



```swift
// ViewController.swift
class ScrollViewController: UIViewController {

  @IBOutlet weak var scrollView: UIScrollView!
    
  override func viewDidLoad() {
    super.viewDidLoad()

      // Do any additional setup after loading the view.
    NotificationCenter.default.addObserver(self, selector: #selector(ViewController.keyboardWillShow), name: UIResponder.keyboardWillShowNotification, object: nil)
    
    NotificationCenter.default.addObserver(self, selector: #selector(ViewController.keyboardWillHide), name: UIResponder.keyboardWillHideNotification, object: nil)
  }
  
  @objc func keyboardWillShow(notification: NSNotification) {
    guard let keyboardSize = (notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue)?.cgRectValue
    else {
      // if keyboard size is not available for some reason, dont do anything
      return
    }

    let contentInsets = UIEdgeInsets(top: 0.0, left: 0.0, bottom: keyboardSize.height , right: 0.0)
    scrollView.contentInset = contentInsets
    scrollView.scrollIndicatorInsets = contentInsets
  }

  @objc func keyboardWillHide(notification: NSNotification) {
    let contentInsets = UIEdgeInsets(top: 0.0, left: 0.0, bottom: 0.0, right: 0.0)
		
    
    // reset back the content inset to zero after keyboard is gone
    scrollView.contentInset = contentInsets
    scrollView.scrollIndicatorInsets = contentInsets
  }
}
```

<br>



The result will look like this :

![scrollUp](https://iosimage.s3.amazonaws.com/2019/69-move-view-when-keyboard-shown/scrollUp.gif)



Pretty nifty!



<script async data-uid="1900ca1895" src="https://fluffy-es.ck.page/1900ca1895/index.js"></script>



## Further Reading

[Apple official documentation on Keyboard management](https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/KeyboardManagement/KeyboardManagement.html)









