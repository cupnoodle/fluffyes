# How to use scroll view in Interface Builder / Storyboard (Xcode 11)

## Structure of scrollview

Scrollview works by having a scrollable content area, like this:  

![scrollviewStructure](/Users/soulchild/Repository/autolayout/images/8/scrollviewStructure.png)



In order for scrollview to work in Auto Layout, **scrollview must know its scrollable content** (scrollview content) **width** and **height** , and also the **frame (X, Y , Width, Height) of itself**, ie. where should the superview place scrollview and what size.



Xcode will show you the error "ambiguous scrollable content width / height" if Auto Layout doesn't know what is the width and height for the **scrollable content** inside scrollview.



## Content Layout guide and Frame Layout guide

In Xcode 11, Apple has introduced **Content Layout Guide** and **Frame Layout Guide** for Scroll View in Interface Builder, which makes it easier to design UI inside Scroll View. This is enabled by default.



![scrollViewLayoutGuide](https://iosimage.s3.amazonaws.com/2019/68-scrollview-xcode11/contentLayoutGuide.png)



To layout a scrollview correctly, we will need two set of constraints : 

1. **Constraints that set the size and position of the scroll view relative to its superview**. (We did this at the start of this chapter by putting top/leading/trailing/bottom constraint from the scrollview to its superview)
2. **Constraints that set the size of the scrollable content area inside the scrollview**, so the scroll view can know the scrollable content dimension and render correctly.



The **Frame Layout guide** relates to the position (x, y) and size (width, height) of the scrollview itself, relative to the superview it was placed in.



![Frame Layout guide](https://iosimage.s3.amazonaws.com/2019/68-scrollview-xcode11/frameLayoutGuide.png)



The **Content Layout guide** relates to the size of the scrollable content area inside the scroll view.

![content layout guide](https://iosimage.s3.amazonaws.com/2019/68-scrollview-xcode11/contentLayoutGuide.png)







