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



The **Bundle.main.localizedString** function will search for the value of the specified key in **forKey** parameter, and return it if it exist. If no such key exist in the strings file, the function will return the value passed into the **value** parameter. The **table** parameter specifies the filename of the strings file to search, in the example above, it will search the key/value pair in "Localizable.strings" file.





// scheme > language

// preferred language



// app name

// NSLocationUsage





// change app language using settings app

// change app language inside app, without settings