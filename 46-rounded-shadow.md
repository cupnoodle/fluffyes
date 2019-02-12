# Implementing rounded corner image view with shadow

Since iOS 11, the App Store app has adopted card UI like this :

![Card UI](https://iosimage.s3.amazonaws.com/2019/46-rounded-shadow/AppStoreCard.png)



The card-like UI has rounded corner and a light drop shadow beneath it, how can we achieve it?



Say we have a image view like this : 

![imageView](https://iosimage.s3.amazonaws.com/2019/46-rounded-shadow/imageView.png)



## Implementing shadow

To add a drop shadow, we can modify the shadowColor, shadowOffset, shadowRadius and shadowOpacity properties of the image view's layer :

```swift
// inside viewDidLoad

// the color of the shadow
imageView.layer.shadowColor = UIColor.darkGray.cgColor

// the shadow will be 5pt right and 5pt below the image view 
// negative value will place it on left / above of the image view
imageView.layer.shadowOffset = CGSize(width: 5.0, height: 5.0)

// how long the shadow will be. The longer the shadow, the more blurred it will be
imageView.layer.shadowRadius = 25.0

// opacity of the shadow
imageView.layer.shadowOpacity = 0.9
```

<br>



After implementing the code above, here's how the image view looks like : 

![Shadow](https://iosimage.s3.amazonaws.com/2019/46-rounded-shadow/shadow.png)



Looking good! In the next section, we will look into how to implement corner radius separately.



## Implementing rounded corner

To implement rounded corner, we can change the cornerRadius property of the image view layer like this : 

```swift
imageView.layer.cornerRadius = 25.0
```

<br>

Build and run the app, then we will get this output : 

![image view](https://iosimage.s3.amazonaws.com/2019/46-rounded-shadow/imageView.png)



Wait, why the image view doesn't get rounded? 🤔

When we set the layer.cornerRadius to 25.0 pt, we are actually setting the bounds of the image view like this : 

![imageView bounds](https://iosimage.s3.amazonaws.com/2019/46-rounded-shadow/bounds.png)

The bounds are rounded, but the image is not following the boundary (bounds) and extended outside the boundary.



To solve this, we can set the **clipsToBounds** property to true so that the image will be confined inside the boundary.

```swift
imageView.layer.cornerRadius = 25.0
imageView.clipsToBounds = true
```

<br>



Now the image view is rounded, 🙌!

![imageView rounded](https://iosimage.s3.amazonaws.com/2019/46-rounded-shadow/rounded.png)





 In the next section, we will look into how to implement both corner radius and shadow together.



## Issue with implementing rounded corner with shadow

Since we now know how to implement rounded corner and shadow, let's combine them and we shall get the App Store card-like UI .



```swift
imageView.layer.cornerRadius = 25.0
imageView.clipsToBounds = true

imageView.layer.shadowColor = UIColor.darkGray.cgColor
imageView.layer.shadowOffset = CGSize(width: 5.0, height: 5.0)
imageView.layer.shadowRadius = 25.0
imageView.layer.shadowOpacity = 0.9
```

<br>

Build and run the app and we will get an output like this : 

![rounded corner only](https://iosimage.s3.amazonaws.com/2019/46-rounded-shadow/rounded.png)



Wait... where did the shadow go? Why combining rounded corner and shadow makes the shadow go away?! 



Remember the **bounds** and **clipsToBounds = true** we mentioned earlier? When we set clipsToBounds = true, the content (image) will be confined inside the rounded bounds. As the shadow is outside the bounds, it got clipped out :



![clipsToBounds](https://iosimage.s3.amazonaws.com/2019/46-rounded-shadow/clipsToBounds.png)

Notice the shadow got clipped after setting clipsToBounds = true.



As we need clipsToBounds = true to make the image become rounded, how can we achieve both rounded corner and shadow effect for the image? 🤔



There's many solutions (some require drawing custom layer ) to this, one of the straightforward solution would be using multiple views (1 view for shadow, 1 view for rounded corner) to achieve the same effect. We will look into this method in the next section.



## Implementing rounded corner with shadow 



// show combined, but shadow somehow gone because of clipping



// straightforward solution : use two views





Lego brick photo by [Iker Urteaga](https://unsplash.com/photos/TL5Vy1IM-uA?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/game?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)