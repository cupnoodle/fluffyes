# Introduction to Dynamic Type support (tutorial)

Table of contents :

1. [Default dynamic type text styles](#default)
2. [Use Accessbility Inspector to speed up testing different text size](#inspector)
3. [Scaling custom fonts](#custom)
4. [Adapting label to larger font size](#adapt)
5. [Detecting text size changes](#detect)
6. [Content Size Category (List of different text sizes)](#category)
7. [Adjusting other UI elements on text size changes](#other)
8. [Further Reading](#further)





Apple introduced dynamic type in iOS 7 to allow users to specify their preferred text size in the Settings app. App that supports dynamic type will be able to adjust text size based on the user preferred text size.



Settings app > General > Accessibility > Larger Text.

![accessbility text size](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/accessiblityPage.png)



This accessibility feature help users with visual impairment , or weakend vision due to aging to be able to read text on the device. By default, the text size slider is positioned in the center.



According to [a report by PDF Viewer app team](https://pspdfkit.com/blog/2018/improving-dynamic-type-support/), around 27 percent of their users have specified a non-default text size, which is quite significant! If your app doesn't support dynamic type, there might be a quarter of users who might be struggling to use your app due to vision problems!



<span id="default"></span>

## Default dynamic type text styles

One of the easiest way to support dynamic type is to use the preset text styles, simply change your label font to the preset text styles and you will get dynamic type support for free .



![change font](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/changeFont.png)



![change font 2](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/changeFont2.png)



There are a variety of text styles ranging from the smallest **Caption 2** to the largest **Large Title** to suit your layout.



Here is how they look on the default text size settings : 

![normal size](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/normalSize.png)



And in larger text size settings : 

![larger size](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/largerSize.png)





By default, the preset text styles font size are set when the app launches, meaning if the user change the settings font size in the mid of using your app, they need to quit and relaunch your app to see the updated text size. 



To support auto text size adjustment (ie. the app text size changes automatically when user adjust the text size in Settings app, without having to relaunch your app), simply check the "**Automatically Adjusts Font**" checkbox in the Attribute inspector.



![auto adjust font](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/autoAdjustFont.png)



And the font size will auto adjust with the Settings text size without needing to relaunch : 

![auto adjustment magic](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/realtimeFont.gif)





Not a fan of using Storyboard / Interface builder? To support dynamic type programmatically, we can use the **preferredFont(forTextStyle:)** method of UIFont.

```swift
// set the font text style to large title
label.font = UIFont.preferredFont(forTextStyle: .largeTitle)
```

<br>

To support auto adjustment for text size changes (same as the "Automatically Adjusts Font" checkbox), set the **adjustsFontForContentSizeCategory** to true.

```swift
// set the font to auto adjust to text size changes in Settings app
label.adjustsFontForContentSizeCategory = true
```

<br>



<span id="inspector"></span>

## Use Accessbility Inspector to speed up testing different text size

It can be quite tedious to keep switching between Settings app and your app to adjust for different text size and test. To speed up this process, we can use the **Accessbility Inspector** in the Xcode developer tools section.

![open accessibility inspector](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/menuAccessbilityInspector.png)



In the accessibility inspector, select the simulator / device you are using, then click the settings icon, and  you can adjust the text size easily by dragging the font size slider (default size is 4th tick from left).

![inspector font size](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/fontSizeInspector.png)



Remember to check the "**Automatically Adjusts Font**" checkbox or set **adjustsFontForContentSizeCategory** to true for this to work.



<span id="custom"></span>

## Scaling custom fonts

As the preset text styles use System Font, what if we want to use a custom font, say Avenir Next and support dynamic type at the same time?



In iOS 11, Apple has introduced [UIFontMetrics](https://developer.apple.com/documentation/uikit/uifontmetrics) class to let us support dynamic type on custom font easily.



To support dynamic type on custom font, we can use the UIFontMetrics to bind a certain text style for a label font like this : 

```swift
// Yes, I know using force unwrapping is bad

let font = UIFont(name: "AvenirNext-Regular", size: 18.0)!
label.font = UIFontMetrics(forTextStyle: .body).scaledFont(for: customFont)
```

<br>



This will make the label's font scale accordingly to the **percentage increase** of the **body** text style. If the body text style point size increases 35% (eg: 17pt to 23pt) when user switch from the default text size to the largest text size, UIFontMetrics will return the AvenirNext font with 35% larger size, ie. 18 * 1.35 = 24pt.



Even though UIFontMetrics has done most of the work for us, we still need to decide what font (and size) to use for each of the text style on the default text size (eg: AvenirNext Regular size 18.0 for the default text size of body text style). To simplify this process, I recommend following [Keith Harrison's Plist approach](https://useyourloaf.com/blog/using-a-custom-font-with-dynamic-type/) on storing different fonts for different text styles.



<span id="adapt"></span>

## Adapting label to larger font size

You might used to only set the top and leading constraints for a label like this : 

![leading and top constraints](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/leadingTopOnly.png)



As the label doesn't have a trailing constraint, it might expand over the available screen size if user selects a larger text size, causing the user unable to see the full text : 

![oops](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/oops.png)



To solve this, we can add a trailing constraint to the label to ensure it won't expand beyond the screen, and set its **Lines** properties to **0** so that it can wrap the text to next line when there is not enough space in the current line.

![label solution](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/labelSolution.png)



After adding trailing constraint and setting number of lines to 0 , it looks good on larger text size like this : 



![looks better](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/looksBetter.png)



<span id="detect"></span>

## Detecting text size changes

Sometimes we might want to perform certain action when user adjusted the text size in the Settings app, we can do so by implementing the **traitCollectionDidChange(_ previousTraitCollection:)** method. This method will be called when user return back to our app after changing text size.

```swift
// YourViewController.swift

class YourViewController : UIViewController {
  override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    // perform action here
  }
}
```

<br>



<span id="category"></span>

## Content Size Category (List of different text sizes)

We can get the current selected text size by using the UIViewController.traitCollection.**preferredContentSizeCategory** property . This property will return the current text size (UIContentSizeCategory), with the default text size being **.large** (ie. the middle option in the text size slider) .



```swift
// YourViewController.swift

class YourViewController : UIViewController {
  override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    // perform action here when user changes the text size
    
    if self.traitCollection.preferredContentSizeCategory == .extraExtraExtraLarge {
      // perform action here when user changes the text size to the largest (non-accessbility)
    }
  }
}
```

<br>



Here's some UIContentSizeCategory values relative to the text size slider : 

![regular sizes](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/regularTextSizes.png)



![accessibility text sizes](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/accessibilityTextSizes.png)



You can view the full range of [UIContentSizeCategory in Apple's documentation](https://developer.apple.com/documentation/uikit/uicontentsizecategory).



<span id="other"></span>

## Adjusting other UI elements on text size changes

Often, there will be a mix of images and labels in a layout, sometimes we might want an image to grow in size following the text size increase. Similar to custom fonts, we can use the **UIFontMetrics** class, but this time with the method **.scaledValue(for:)** to scale the image size according to the percentage increase of the text size.



Say we want to bind the image height to the headline text style, and the default value for image height is 75.0, we can bind it like this : 

```swift
// YourViewController.swift

// the height constraint for the avatar image view
@IBOutlet weak var avatarHeightConstraint: NSLayoutConstraint!

override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
  // if the headline text size increase by 35%, the avatar height will increase by 35% too.
  let adjustedHeight = UIFontMetrics(forTextStyle: .headline).scaledValue(for: 75.0)
  avatarHeightConstraint.constant = adjustedHeight
}

```

<br>



Here's the example result : 



![adaptiveImage](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/adaptiveImage.gif)



// CTA sample Xcode project for the demo used in this post



<span id="further"></span>

## Further Reading

You can check all the point size for different text styles at different system text size on [Apple Human Interface Guideline's typography page](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/typography/).



[Detailed tutorial on supporting dynamic type for custom fonts](https://useyourloaf.com/blog/using-a-custom-font-with-dynamic-type/) by Keith Harrison.



[Building Apps with Dynamic Type -  WWDC 2017](https://developer.apple.com/videos/play/wwdc2017/245/)