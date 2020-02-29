# Localization tutorial (add additional language support) 



Localization means making your app support additional languages. Usually we will start with english as the base language as default, then slowly adding more language support on top of it.



![translation demo](https://iosimage.s3.amazonaws.com/2020/72-localization/translate.png)



Before starting localization of your app, make sure the project has "Use Base Internationalization" checked. This is checked by default when you start a new project in Xcode 11.



![use base internationalization checked](https://iosimage.s3.amazonaws.com/2020/72-localization/useBaseInternationalization.png)



## What does "Use Base Internationalization" mean?

When you create a new iOS project in Xcode 11, Xcode will auto generate the resources for the default language (which is the "base" language), like Main.storyboard and LaunchScreen.storyboard, and place them into the "Base.lproj" folder. The "base" means other language added later on will use this as base to refer to.



![whats is base](https://iosimage.s3.amazonaws.com/2020/72-localization/whatIsBase.png)



## Adding new language

Now that we have base localization set, we can proceed to add more language by clicking the "+" button below : 

![add new language](https://iosimage.s3.amazonaws.com/2020/72-localization/addNewLanguage.png)





// scheme > language

// preferred language