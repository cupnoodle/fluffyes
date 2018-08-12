# Replicating Spotify's Now Playing UI using Auto Layout - Part 1 / 2

In this post, we will breakdown and analyze the Now Playing screen of Spotify app, and try to replicate it  using Auto Layout. This post assume you have some experience working with [Auto Layout](https://fluffy.es/making-sense-of-auto-layout/). 

The playback button icons used in this post are from [Font Awesome](https://fontawesome.com), you can use [fa2png.io](http://fa2png.io) to generate .png images from the font awesome icons.



Here's the Now Playing screen of Spotify viewed in iPhone 8 screen size: 

![Spotify UI](https://iosimage.s3.amazonaws.com/2018/24-spotify/spotifyUI.png)



The scrollable album arts is a collection view, song name and artist name are labels, there's five button (shuffle, previous, play/pause, next, repeat), the song progress bar is a slider. The left two and right two buttons are grouped in stack views, this will be explained more later. 



![Spotify UI Analyze](https://iosimage.s3.amazonaws.com/2018/24-spotify/spotifyAnalyze.png)



The vertical distance of each UI elements are shown using the blue arrow. Feel free to tweak the value of the vertical constraint until it feels right for you.


## Album Art Collection View

Let's say each album art is a collection view cell itself, each cell is a square , meaning its width is equal to height, this means a ratio of 1:1 .

![Collection Cell](https://iosimage.s3.amazonaws.com/2018/24-spotify/collectionCell.png)



To make it simple, assume the collection view containing the cell has the same height as the cell itself, with same width but with added spacing on the left (34 pt) and right (34 pt). This means the collection view has an aspect ratio of 1:1 + 34 +34 = 1:1 + 68.

![Collection View](https://iosimage.s3.amazonaws.com/2018/24-spotify/collectionView.png)



```
1 * width = 1 * height + 34 +34
width = height + 68
height = width - 68
```

<br>

The collection view has a top constraint to the top edge of safe area, leading constraint with value 0, trailing constraint with value 0. The leading and trailing constraint means that the collection view's width is equal to the screen width.



Since there are distances from the album art to the screen edge (left and right), and the album art is a square, the height of the collection view will be 68 pt lesser than its width.



We can set a ratio constraint on the collection view like this :  

![Aspect ratio constraint](https://iosimage.s3.amazonaws.com/2018/24-spotify/aspectRatio1.png)



![Aspect Ratio 2](https://iosimage.s3.amazonaws.com/2018/24-spotify/aspectRatio2.png)



## Album Art Collection View Cell

After creating the collection view, drag a collection view cell into the collection view if there isn't one already : 

![Collection View Cell](https://iosimage.s3.amazonaws.com/2018/24-spotify/addCell.png)



After that, drag an image view into the cell and add constraints to make the image has the same size as the cell. Create a top, leading, trailing and bottom constraints with value 0 for the image view.

![Cell Image Constraint](https://iosimage.s3.amazonaws.com/2018/24-spotify/cellImageConstraint.png)



Next, set the imageview's image to your favorite album art picture! 😆



## Play button

Say you are using the play button from Font Awesome like this :  

![Play button](https://iosimage.s3.amazonaws.com/2018/24-spotify/playButton.png)



To make it simple, we set a fixed width (74pt) and height (74pt) for the Play button and center it horizontally to the parent view.



To add a surrounding circle to the play button, we can use the cornerRadius and borderWidth property of the button's layer.

```swift
// the circle around the play button
playButton.layer.cornerRadius = playButton.frame.size.height / 2.0
playButton.layer.borderWidth = 2.0
playButton.layer.borderColor = UIColor.white.cgColor
```

<br>

Hmmm, now the play button image seems too big and too close to the surrounding circle :
![too big](https://iosimage.s3.amazonaws.com/2018/24-spotify/tooBig.png)



We can adjust the **Image Insets** of the button in Size inspector tab to add padding to the image : 
![Image insets](https://iosimage.s3.amazonaws.com/2018/24-spotify/imageInsets.png)



The image insets means the spacing from the image to the button edge. With the above values, we will set 20 pt padding on top, left, bottom and right for the image inside button.



After adding insets, the play button looks better now with padding :

![circled play button](https://iosimage.s3.amazonaws.com/2018/24-spotify/circledPlayButton.png)



## Playback buttons and stack view

The play button is located in the center (using the align horizontal center constraint), and there's other buttons beside it. At first glance, using fixed leading / trailing constraint between each button seems to do the job : 

![Spacing between buttons](https://iosimage.s3.amazonaws.com/2018/24-spotify/spacing.png)



These constraints looks fine on an iPhone SE screen size, but when viewed on iPhone 8 plus, the left and right edge of the screen feels empty as it have too much blank space.

![Extra spacing](https://iosimage.s3.amazonaws.com/2018/24-spotify/extraSpacing.png)



To reduce the blank space, we will have to distribute the button evenly, meaning they occupy a certain proportion of the width of the screen size. As the screen width increase, they should occupy more width so that there wouldn't be a chunk of blank space.




To distribute button evenly, we can use a stack view and set its Distribution property to **Fill Equally**. Then we place placeholder views inside the stackview and each of the placeholder view has the same width. Next we place the button inside the placeholder view, and horizontally center the button to the placeholder view.

![Stack view](https://iosimage.s3.amazonaws.com/2018/24-spotify/buttonStackView.png)



Repeat this process for the left part (shuffle and previous buttons). Now you should have an evenly distributed buttons. But you might find the image on the button stretches, this is because the content mode of image inside a button is "Scale to Fill" by default. To resolve the stretching issue, we can set the content mode to "Aspect Fit" using code :

```swift
// make the image on these button become aspect fit so it won't stretch
shuffleButton.imageView?.contentMode = .scaleAspectFit
previousButton.imageView?.contentMode = .scaleAspectFit
forwardButton.imageView?.contentMode = .scaleAspectFit
repeatButton.imageView?.contentMode = .scaleAspectFit
```

<br>



This is how the playback buttons looks like in iPhone SE and iPhone 8 plus using stack view:  

![Evenly distributed buttons](https://iosimage.s3.amazonaws.com/2018/24-spotify/buttonsCompare.png)



## Progress Slider

A default slider looks like this: 
![default slider](https://iosimage.s3.amazonaws.com/2018/24-spotify/defaultSlider.png)



You can customize the left bar color and right bar color with the Min Track and Max track color property.

![minMax Color](https://iosimage.s3.amazonaws.com/2018/24-spotify/minMaxColor.png)



As the default circle size is too large for the song progress slider, we can replace it with a [smaller circle](https://iosimage.s3.amazonaws.com/2018/24-spotify/sliderThumb.png) (the circle is called as "Thumb"). You will need to prepare an image containing a smaller circle for this.

```swift
// set image for the button of the slider, when not selected, its small
progressSlider.setThumbImage(UIImage(named: "sliderThumb"), for: .normal)
```

<br>



If you have noticed, when you are selecting the slider in Spotify, the thumb become larger and has another layer of half-transparent circle around it. 

![selectedStateSlider](https://iosimage.s3.amazonaws.com/2018/24-spotify/selectedState.png)


The easiest way to replicate this is to prepare an [image](https://iosimage.s3.amazonaws.com/2018/24-spotify/sliderThumbSelected.png) for it. And set a different thumb image for the selected state.

```swift
// when selected, use a larger image
progressSlider.setThumbImage(UIImage(named: "sliderThumbSelected"), for: .highlighted)
```

<br>



## Current result

![Part 1 Result](https://iosimage.s3.amazonaws.com/2018/24-spotify/part1result.png)

It kinda looks good now! Just that the collection view cell doesn't have spacing and doesn't have the zooming / fade out effect when scrolled. We will add these effect into the collection view in part 2.
