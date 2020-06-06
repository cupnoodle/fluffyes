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



We can create a struct which conform to **UIViewControllerRepresentable**, and return this view controller in the **makeUIViewController(context: Context)** method.  This will wrap the UIViewController in a SwiftUI View.



We can leave the updateUIViewController function empty, as we don't have SwiftUI state that will update the view controller in this case.



Next, we will use this struct inside another struct which conforms **PreviewProvider** protocol, to generate the preview : 



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



With UIViewControllerRepresentable and PreviewProvider implemented, we now can view the preview. If the preview pane doesn't show, press **option** + **command** + **enter** (or from the top menu **Editor** > **Canvas**) to open the preview pane.



![preview screenshot](https://iosimage.s3.amazonaws.com/2020/79-xcode-previews-uikit/previewScreenshot.png)



Now when you make any changes to the view controller code, you can see it reflect (almost) immediately on the preview! 🚀



It can be repetitive to implement the UIViewControllerRepresentable struct for each of the View Controllers, we can create an extension for this to eliminate this repeating work. 



## UIViewControllerRepresentable extension

Let's create new Swift File for this extension : 

![new file 1](https://iosimage.s3.amazonaws.com/2020/79-xcode-previews-uikit/new_file_1.png)



![new file 2](https://iosimage.s3.amazonaws.com/2020/79-xcode-previews-uikit/new_file_2.png)



Name it as "**UIViewController+Preview.swift**".

![new file 3](https://iosimage.s3.amazonaws.com/2020/79-xcode-previews-uikit/new_file_3.png)



Then put in the extension code like below : 

```swift
//UIViewController+Preview.swift

import UIKit

#if DEBUG
import SwiftUI

@available(iOS 13, *)
extension UIViewController {
    private struct Preview: UIViewControllerRepresentable {
        // this variable is used for injecting the current view controller
        let viewController: UIViewController

        func makeUIViewController(context: Context) -> UIViewController {
            return viewController
        }

        func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
        }
    }

    func toPreview() -> some View {
        // inject self (the current view controller) for the preview
        Preview(viewController: self)
    }
}
#endif
```



With this extension, we just need to add the PreviewProvider confirming struct below each of the view controller, and use **ViewController.toPreview()** for the preview view.



```swift
// InfoViewController.swift

import UIKit

class InfoViewController: UIViewController {
  ...
}

#if DEBUG
import SwiftUI

@available(iOS 13, *)
struct InfoVCPreview: PreviewProvider {
    
    static var previews: some View {
        // view controller using programmatic UI
        InfoViewController().toPreview()
    }
}
#endif

```



If you are using Storyboard for the view controller, you can instantiate it using the **Storyboard ID**.



Ensure that you have set a unique storyboard ID for your view controller : 

![storyboard ID](https://iosimage.s3.amazonaws.com/2020/79-xcode-previews-uikit/storyboard_id.png)



Then in the PreviewProvider, instantiate the view controller from the storyboard : 

```swift
#if DEBUG
import SwiftUI

@available(iOS 13, *)
struct ProfileVCPreview: PreviewProvider {
    static var previews: some View {
        // Assuming your storyboard file name is "Main" 
        UIStoryboard(name: "Main", bundle: nil).instantiateViewController(identifier: "ProfileViewController").toPreview()
    }
}
#endif
```



## Previewing multiple devices

It would be handy if we can preview how our UI looks like in multiple devices at once. We can specify an array of devices and create preview of our UI in each of the device like this : 

```swift

```






