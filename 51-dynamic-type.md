# Introduction to supporting Dynamic Type

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





One thing to note is that even for the same text styles (eg: Large Title) with the same text size selected (eg: the default middle text size) in Settings app, the text size displayed in app will still be different across different devices, 



![different devices](https://iosimage.s3.amazonaws.com/2019/51-dynamic-type/differentSizes.png)



Notice that the Large Title text size is smaller in iPhone SE compared to iPhone 8 Plus although both of them have used the same default (middle) text size in the Settings app.



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

