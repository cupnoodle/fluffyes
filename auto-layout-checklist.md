# Auto Layout not working Checklist



You are stuck on designing UI as it doesn't work like the way you wanted it to and some views simply doesn't appear on screen when you run the app. You have spent 3 hours making all the red lines turn into blue in Interface Builder by now.

Here is a checklist to troubleshoot why it is not working :

1. If you are adding UIView through code, have you set `view.translatesAutoresizingMaskIntoConstraints = false ` ?
2. Did you put enough constraints so that the Auto Layout is able to [calculate each of the views position and size](https://fluffy.es/how-auto-layout-calculates-view-position-and-size/) ?
3. If you are adding constraint using code, did you activate the constraints AFTER you place the view in parent view ? eg:
```swift
// add the programmatically created view to its parent before activating constraints
parentView.addSubview(childView)
...
NSLayoutConstraint.activateConstraints([childBottomConstraint])
```

<br>

4. Is there any hidden conflicting constraint? Try to switch between different device in the Interface Builder and see if theres any red line with numbers  ![Device List](https://iosimage.s3.amazonaws.com/2018/8-conflicting-constraint/deviceList.png)
  <br>
  <br>