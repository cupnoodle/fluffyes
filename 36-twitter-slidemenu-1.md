# Replicating Twitter-like Slide Menu - Part 1



In this post, we will breakdown, analyze the Slide Menu mechanism of Twitter app, and try to replicate it using Auto Layout and Container View. This post assume you have some experience working with [Auto Layout](https://fluffy.es/making-sense-of-auto-layout/).



Main screen of Twitter app consist of a tab view controller. If you swipe right, a side menu will appear.

![side menu and tab bar](https://iosimage.s3.amazonaws.com/2018/36-twitter-slidemenu-1/sideMenuTabBar.png)



When the side menu is open, both side menu view and tab bar view appear in the same screen. The screen (which is a view controller) contain both of the side menu view controller and tab bar view controller : 









