Howdy {{ subscriber.firstname }},<br />
<br />
Following the article you have read previously (<a href="https://fluffy.es/saving-custom-object-into-userdefaults"><span>https://fluffy.es/saving-custom-object-into-userdefaults</span></a>),&nbsp;<br />
&nbsp;
<h2>Why PropertyListEncoder / PropertyListDecoder is used for UserDefaults?</h2>
You might ask, why is it called "PropertyListEncoder" when you are encoding the object to save in UserDefaults, shouldn't it be called "UserDefaultsEncoder" or something?<br />
<br />
This is because when you are saving stuff into UserDefaults, you are <strong>actually saving it to a .plist</strong> (property list) file in your app folder (The folder that contains additional data of your app after installation). The filename of the .plist usually will be `[bundleIdentifier].plist`. In case you forgot whats a bundle identifier, it's this: &nbsp;<br />
&nbsp;&nbsp;<img alt="bundle identifier" src="https://fluffypublic.s3.amazonaws.com/email/1-userdefaults/bundleIdentifier.png" /><br />
<br />
If you open the .plist file using a text editor, you will see something like this :&nbsp;<br />
&nbsp; <img alt="plist file of user defaults" src="https://fluffypublic.s3.amazonaws.com/email/1-userdefaults/plist.png" /><br />
&nbsp;
<p>The&nbsp;<code>&lt;key&gt;player&lt;/key&gt;</code>&nbsp;means the&nbsp;<code>player</code>&nbsp;key we have set previously, and its value type is Data as indicated with&nbsp;<code>&lt;data&gt; &lt;/data&gt;</code>. The text inside the&nbsp;<code>&lt;data&gt; &lt;/data&gt;</code>&nbsp;tags is the encoded value using PropertyListEncoder.</p>

<p>To access the app folder of iOS simulators on your Mac, you can use this code:</p>

<pre>
// the #if ensure that this code only runs in iOS Simulator but not real device
#if arch(i386) || arch(x86_64)
    if let documentsPath = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first?.path {
    print("Documents Directory: \(documentsPath)")
    }
#endif</pre>
This will output the path of the documents folder inside the app folder:&nbsp;&nbsp;<br />
<br />
<img alt="Path output" src="https://fluffypublic.s3.amazonaws.com/email/1-userdefaults/pathOutput.png" /><br />
<br />
Simply remove the 'Documents' at the back of the string to get the app folder path. The app folder path should look like this:
<pre>
/Users/[Your Mac Username]/Library/Developer/CoreSimulator/Devices/[Device UUID]/data/Containers/Data/Application/[App UUID]/</pre>
<br />
You can copy this path and open it in Finder using the keyboard shortcut&nbsp;<strong>Command</strong>+&nbsp;<strong>Shift</strong>&nbsp;+&nbsp;<strong>G</strong>&nbsp;.<br />
&nbsp; <img alt="Go to folder" src="https://fluffypublic.s3.amazonaws.com/email/1-userdefaults/goto.png" /><br />
<br />
Once you are in the app folder path, the .plist file is located in Library -&gt; Preferences folder.<br />
&nbsp; <img alt="Plist location" src="https://fluffypublic.s3.amazonaws.com/email/1-userdefaults/plistlocation.png" /><br />
<br />
If you double click it, most likely Xcode will open this file for you.<br />
Here you can see and manually edit UserDefaults value!<br />
<br />
To edit UserDefaults value in a real device, you will need some software like&nbsp;<a href="http://www.i-funbox.com/">iFunBox</a>&nbsp;.<br />
<br />
Hope you have learned how UserDefaults stores the data in this email!<br />
<br />
Regards,<br />
<br />
Axel