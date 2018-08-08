# Replicating Spotify's Now Playing UI using Auto Layout

In this post, we will breakdown and analyze the Now Playing screen of Spotify app, and try to replicate it  using Auto Layout. This post assume you have some experience working with Auto Layout, if not, check out the [Making Sense of Auto Layout series](https://fluffy.es/making-sense-of-auto-layout/)!



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

The play button is located in the center (use the align horizontal center constraint), and there's other buttons beside it.











