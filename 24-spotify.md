# Replicating Spotify's Now Playing UI using Auto Layout

In this post, we will breakdown and analyze the Now Playing screen of Spotify app, and try to replicate it  using Auto Layout. This post assume you have some experience working with [Auto Layout](https://fluffy.es/making-sense-of-auto-layout/). The playback button icons used in this post are from [Font Awesome](https://fontawesome.com).



Here's the Now Playing screen of Spotify viewed in iPhone 8 screen size: 

![Spotify UI](https://iosimage.s3.amazonaws.com/2018/24-spotify/spotifyUI.png)



The scrollable album arts is a collection view, song name and artist name are labels, there's five button (shuffle, previous, play/pause, next, repeat), the song progress bar is a slider. The left two and right two buttons are grouped in stack views, this will be explained more later. 



![Spotify UI Analyze](https://iosimage.s3.amazonaws.com/2018/24-spotify/spotifyAnalyze.png)



## Album art Collection View

The collection view has a top constraint to the top edge, leading constraint with value 0, trailing constraint with value 0. The leading and trailing constraint means that the collection view's width is equal to the screen width.



The tricky one is the aspect ratio constraint, the 1:1 ratio means that the width is equal to the height and vice versa. 1:1 ratio + 68 means that 

```
1 * width + 68 = 1 * height
width + 68 = height
height = width - 68
```



Since there are distances from the album art to the screen edge (left and right), and the album art is a square, the height of the collection view will be 68 pt lesser than its width.



# Playback buttons and stack view

The play button is located in the center (using the align horizontal center constraint), and there's other buttons beside it. At first glance, using fixed leading / trailing constraint between each button seems to do the job : 

![Spacing between buttons](https://iosimage.s3.amazonaws.com/2018/24-spotify/spacing.png)



These constraints looks fine on an iPhone SE screen size, but when viewed on iPhone 8 plus, the left and right edge of the screen feels empty as it have too much blank space.

![Extra spacing](https://iosimage.s3.amazonaws.com/2018/24-spotify/extraSpacing.png)



To reduce the blank space, we will have to distribute the button evenly, meaning they occupy a certain proportion of the width of the screen size. As the screen width increase, they should occupy more width so that there wouldn't be a chunk of blank space.



To make it simple, we set a fixed width (74pt) and height (74pt) for the Play button and center it horizontally to the parent view.




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



This is how the playback buttons looks like in iPhone SE and iPhone 8 plus using stack view:  

![Evenly distributed buttons](https://iosimage.s3.amazonaws.com/2018/24-spotify/buttonsCompare.png)



## Progress Slider























