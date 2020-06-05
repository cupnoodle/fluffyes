# Use Xcode Previews with UIKit

**Prerequisite**: You will need macOS Catalina (10.15)+ and Xcode 11+ to run Xcode Previews.



![demo](https://iosimage.s3.amazonaws.com/2020/79-xcode-previews-uikit/demo.gif)



Apple introduced Xcode Previews in WWDC 2019 alongside with SwiftUI, which allow us to view UI changes immediately after each change, instead of needing to recompile it after each UI changes.




You might think that Xcode Previews works for SwiftUI project, but it can work on UIKit's UIViewController and UIView too! This makes coding UI programmatically a lot more attractive now with (almost) instant preview, and you don't have to deal with slow IBDesignables in Storyboard anymore.



This article assumes that your iOS app supports iOS 12 and older, you can remove the **@available(iOS 13, *)** line if your app only supports iOS 13 and newer.



## How Previews work

To enable preview for a SwiftUI view, we need to create a struct which conform to the **PreviewProvider** protocol like this : 



```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        // the SwiftUI View
        ContentView()
    }
}
```



When you create a new SwiftUI View, Xcode will auto generate this block at the bottom of the View code for you.



 Since we are using UIKit's view, we need to wrap our UIKit View Controller inside a SwiftUI View to make it work, using **UIViewControllerRepresentable** protocol, or UIViewRepresentable if you are using UIView instead of UIViewController.



## Wrapping UIViewController inside SwiftUI View

Say you have a View Controller like this :

```swift
// InfoViewController.swift

import UIKit

class InfoViewController: UIViewController {
  ...
}
```



We can create a struct which conform to UIViewControllerRepresentable, and return this view controller in the **makeUIViewController(context: Context)** method. 

```swift
// InfoViewController.swift

import UIKit

class InfoViewController: UIViewController {
  ...
}

#if DEBUG
import SwiftUI

struct InfoVCRepresentable: UIViewControllerRepresentable {
    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
        // leave this empty
    }
    
    @available(iOS 13.0.0, *)
    func makeUIViewController(context: Context) -> UIViewController {
        InfoViewController()
    }
}
```



Next, we will use this struct inside another struct which conforms PreviewProvider protocol, to generate the preview : 

```swift
// InfoViewController.swift

import UIKit

class InfoViewController: UIViewController {
  ...
}

#if DEBUG
import SwiftUI

struct InfoVCRepresentable: UIViewControllerRepresentable {
    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
        // leave this empty
    }
    
    @available(iOS 13.0.0, *)
    func makeUIViewController(context: Context) -> UIViewController {
        InfoViewController()
    }
}

@available(iOS 13.0, *)
struct InfoVCPreview: PreviewProvider {
    static var previews: some View {
       InfoVCRepresentable()
    }
}
#endif
```



With UIViewControllerRepresentable and PreviewProvider implemented, we now can view the preview. If the preview pane doesn't show, press **option** + **command** + **enter** to open the preview pane.

image of preview





to-do: put it into an extension, so all view controllers can use it










