# Introduction to Dynamic Type support (tutorial)

Apple introduced dynamic type in iOS 7 to allow users to specify their preferred text size in the Settings app. App that supports dynamic type will be able to adjust text size based on the user preferred text size.



Settings app > General > Accessibility > Larger Text.

![accessbility text size](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/accessiblityPage.png)



This accessibility feature help users with visual impairment , or weakend vision due to aging to be able to read text on the device. By default, the text size slider is positioned in the center.



According to [a report by PDF Viewer app team](https://pspdfkit.com/blog/2018/improving-dynamic-type-support/), around 27 percent of their users have specified a non-default text size, which is quite significant! If your app doesn't support dynamic type, there might be a quarter of users who might be struggling to use your app due to vision problems!



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



## Use Accessbility Inspector to speed up testing different text size

It can be quite tedious to keep switching between Settings app and your app to adjust for different text size and test. To speed up this process, we can use the **Accessbility Inspector** in the Xcode developer tools section.

![open accessibility inspector](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/menuAccessbilityInspector.png)



In the accessibility inspector, select the simulator / device you are using, then click the settings icon, and  you can adjust the text size easily by dragging the font size slider (default size is 4th tick from left).

![inspector font size](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/fontSizeInspector.png)



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





