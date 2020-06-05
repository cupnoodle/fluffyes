# Use Xcode Previews with UIKit

Note: You will need macOS Catalina (10.15)+ and Xcode 11+ to run Xcode Previews.



Apple introduced Xcode Previews in WWDC 2019 alongside with SwiftUI, which allow us to view UI changes immediately after each change, instead of needing to recompile it after each UI changes.




You might think that Xcode Previews works for SwiftUI project, but it can work on UIKit's UIViewController and UIView too! This makes coding UI programmatically a lot more attractive now with (almost) instant preview, and you don't have to deal with slow IBDesignables in Storyboard anymore.



This article assumes that your iOS app supports iOS 12 and older, you can remove the **@available(iOS 13, *)** line if your app only supports iOS 13 and newer.












