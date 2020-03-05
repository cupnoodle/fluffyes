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



After selecting a language, Xcode will ask if you want to create localization for current resource (storyboard / XIBs) :

![create Localization dialog](https://iosimage.s3.amazonaws.com/2020/72-localization/createLocalization.png)



After you click 'Finish' , Xcode will create a Strings file for each of the resources.



For example, say your Main.storyboard has a view controller with two labels like this : 

![storyboard with labels](https://iosimage.s3.amazonaws.com/2020/72-localization/storyboardLabel.png)



If you have included Main.storyboard (checked in the dialog above) and click 'Finish', Xcode will create a 'Main.strings' file (spanish) for translating the labels shown in the storyboard.

![Main strings](https://iosimage.s3.amazonaws.com/2020/72-localization/storyboardStrings.png)



You can change the text of "Good morning" to "Buenos días" in Main.strings, then when you run the app in a device which primary language is set to Spanish, it will show "Buenos días".



I prefer to create a separate Strings file which contain all the text that need to be localized, instead of separating the text based on storyboard files. In this tutorial we will ignore the auto-generated Main.strings file, and creating our own Strings file.



Right click your project folder, select '**New File**' , then scroll down to find '**Strings**', select it and click '**Next**'. 



Then name the file as '**Localizable.strings**' and click 'Create'. Older Xcode versions will automatically create this file for you when you localize the project, recent Xcode version doesn't do this automatically anymore, hence we need to create it manually.



![new file](https://iosimage.s3.amazonaws.com/2020/72-localization/newFile.png)



![new strings file](https://iosimage.s3.amazonaws.com/2020/72-localization/newStringsFile.png)



![Localizable.strings](https://iosimage.s3.amazonaws.com/2020/72-localization/Localizable.png)



Now you have a 'Localizable.strings' created for the base language. We can then add a Spanish version (and other languages) of this file, by clicking the '**Localize**' button on the file :



![localize strings file](https://iosimage.s3.amazonaws.com/2020/72-localization/localizeFile.png)



After clicking 'Localize', Xcode will ask if you want to localize the file, and select a default language for it. Usually I use 'English' as the default language and gradually add more language support to it.

![englishLocalize](https://iosimage.s3.amazonaws.com/2020/72-localization/englishLocalize.png)



Now on the 'Localization' section of the File inspector, we can see the language available for this project (You can add more language by going back to the project settings and click '+'). Check the 'Spanish' (or the language you have added) to add localization support.



![addSpanish](https://iosimage.s3.amazonaws.com/2020/72-localization/addSpanish.png)



After checking the additional language, you can expand the "Localizable.strings" and see there's multiple language version : 

![localizable strings multiple version](https://iosimage.s3.amazonaws.com/2020/72-localization/stringLocalize.png)



When localizing a file, Xcode will create separate language folder for that file. Eg: "**es.lproj**" for spanish ("es"), and "**en.lproj**" for english, etc. The localized file is then put into these folders.



![folder structure](https://iosimage.s3.amazonaws.com/2020/72-localization/folderStructure.png)



![folder structure 2](https://iosimage.s3.amazonaws.com/2020/72-localization/folderStructure2.png)



## Adding and using translated text

To add translation text, we need to input key value pair of each translation in this format:

**"key" = "value";**



For example, the english Localizable.strings file looks like this

![english localizable strings](https://iosimage.s3.amazonaws.com/2020/72-localization/englishText.png)



Then in the spanish Localizable.strings file : 

![spanish localizable string](https://iosimage.s3.amazonaws.com/2020/72-localization/spanishText.png)



Notice that we uses the same key for the same text translation in both file, and each translation is separated by a semicolon (**;**).



To use the translation text, we can call **Bundle.main.localizedString(forKey:, value: , table: ) **:

```swift
class ViewController: UIViewController {
    
    @IBOutlet weak var helloLabel: UILabel!
    @IBOutlet weak var goodMorningLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        
        // return 'Hello' for english, 'Hola' for spanish
        helloLabel.text = Bundle.main.localizedString(forKey: "Hello", value: "Hello", table: "Localizable")
        
        // return 'Good Morning' for english, 'Buenos días' for spanish
        goodMorningLabel.text = Bundle.main.localizedString(forKey: "Good Morning", value: "Good Morning", table: "Localizable")
    }
}
```

<br>



The [**Bundle.main.localizedString**](https://developer.apple.com/documentation/foundation/bundle/1417694-localizedstring) function will search for the value of the specified key in **forKey** parameter, and return it if it exist. If no such key exist in the strings file, the function will return the string passed into the **value** parameter. 



The **table** parameter specifies the filename of the strings file to search, in the example above, it will search the key/value pair in "Localizable.strings" file. If you pass in **nil** for the **table** parameter, it will look for "Localizable.strings" by default.



Now if you run the app in spanish language, you will see the following output : 



![spanish label](https://iosimage.s3.amazonaws.com/2020/72-localization/spanishLabel.png)





You might have seen [**NSLocalizedString()**](https://developer.apple.com/documentation/foundation/nslocalizedstring) function before, it's a shortcut function Apple has provided us. NSLocalizedString take in two parameters, the key and comment, and then it will call Bundle.main.localizedString() by passing the key to **forKey** and **value** parameters, and set the **table** to **nil** , which will search for "Localizable.strings" file by default.

![nslocalizedstring](https://iosimage.s3.amazonaws.com/2020/72-localization/nslocalizedstring.png)



The comment parameter is for your own reference, it doesn't do anything, you can think of it like the comment you place in your code.



We can then replace the Bundle.main.localizedString with the shorter NSLocalizedString like this : 

```swift
 override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.

    helloLabel.text = NSLocalizedString("Hello", comment: "")
    goodMorningLabel.text = NSLocalizedString("Good Morning", comment: "")
}
```

<br>



## Preferred language and how to change them

When user launch your app for the first time, iOS will decide which language to use based on their **preferred language order**, not the iPhone language.



![preferred language order](https://iosimage.s3.amazonaws.com/2020/72-localization/preferredLanguageOrder.jpg)



Your user might set iPhone language to English, but using another language as the most preferred language (the top row of preferred language order).



iOS will choose the **first matching language** from top to bottom in the preferred language list to launch your app. If your app doesn't support all the languages listed in the preferred languages , iOS will try to make guesses to choose the nearest language possible. (eg: your app support Portuguese (Portugal) but the user only have Portuguese(Brazil) and English, then iOS will choose Portuguese (Portugal)).



Source: https://developer.apple.com/videos/play/wwdc2016/201/ , timestamp: 11:02



App-specific preferred language is introduced in iOS 13, this means that user can change preferred language for that specific app. In the Settings app, user can scroll to your app, and change the preferred language for just your app without changing the device preferred languages.



![app specific language](https://iosimage.s3.amazonaws.com/2020/72-localization/appSpecificLanguage.png)



After user change the preferred language for your app and switch back to your app, your app will be relaunched and NSLocalizedString will use the updated language.



![switch language](https://iosimage.s3.amazonaws.com/2020/72-localization/switchLanguage.gif)



It can be quite troublesome to go to the Settings app, and change language just to test your localization every time you update a text.



To shortcut this process, we can set the app to always launch in a language during debug session. To do this, in Xcode, select **Product** > **Scheme** > **Manage Schemes ...**



![manage scheme](https://iosimage.s3.amazonaws.com/2020/72-localization/manageScheme.png)



In the scheme window , double click your app scheme : 

![double click app scheme](https://iosimage.s3.amazonaws.com/2020/72-localization/selectAppScheme.png)



Then in the Run/debug tab, change the "Application Language" to your desired language.

![scheme language](https://iosimage.s3.amazonaws.com/2020/72-localization/selectSchemeLanguage.png)





And now every time you build and run your app from Xcode, it will use that language as the preferred language.



## Detecting the current app language

Say you want to show / hide some UI based on the current app language (not the device language), you can get the current selected app language with **Bundle.main.preferredLocalizations.first** .



```swift
// if the app language is japanese
if(Bundle.main.preferredLocalizations.first == "ja"){
    print("the app is using japanese language")
}
```

<br>



From [Apple's documentation](https://developer.apple.com/documentation/foundation/bundle/1413220-preferredlocalizations), preferredLocalizations will return an array of strings,  containing language IDs for localizations in the bundle. The strings are ordered according to the user's language preferences and **available localizations**. We use the **.first** to get the most preferred language of the app (which the app is using).



You might see some StackOverflow answer suggesting to use **Locale.preferredLanguage.first** for getting the current language, keep in mind that **this will get the first preferred language of the device, not the app**. 



![difference](https://iosimage.s3.amazonaws.com/2020/72-localization/difference.png)





## Localizing App Name and system's message

To localize app name and other text that are defined in the Info.plist file (eg: Camera usage description), we will need to create a **InfoPlist.strings** file. Right click your project folder, select 'New File...' > 'Strings File' , and name it as **InfoPlist.strings**.



![new file](https://iosimage.s3.amazonaws.com/2020/72-localization/newFile.png)



![new strings file](https://iosimage.s3.amazonaws.com/2020/72-localization/newStringsFile.png)



![Localizable.strings](https://iosimage.s3.amazonaws.com/2020/72-localization/infopliststrings.png)



Remember to localize it by clicking the 'Localize' button, 

![localize](https://iosimage.s3.amazonaws.com/2020/72-localization/localizeFile2.png)



Then check all the language you want to localize in the "**Localization**" section.



For app name, the key is "**CFBundleDisplayName**"









// app name

// NSLocationUsage





// change app language using settings app

// change app language inside app, without settings