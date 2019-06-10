# How to store image into CoreData

*Note: Remember to check 'allows external storage' in Core Data to avoid potential performance issue. Read below for more explanation.*



If your app need to save a lot of data which fits a certain schema (eg: a person class which have property first name, last name and age), Core Data can be a good choice for it, it can save Strings, Integer, Double, Boolean, Date etc and even Binary Data.



![Core Data attribute type](https://iosimage.s3.amazonaws.com/2019/56-store-image-coredata/dataType.png)



As there's no direct way to save image into the attribute, we have to convert the image into Data type. Say you have an image retrieved from Photo Album / Camera like this : 

```swift
extension ViewController : UIImagePickerControllerDelegate {
  // retrieved image from photo album / camera
  func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
        
    picker.dismiss(animated: true, completion: nil)
        
    guard let chosenImage = info[UIImagePickerController.InfoKey.editedImage] as? UIImage else {
      return
    }
     
  }
}
```

<br>



We can turn the UIImage into png data like this :

```swift
let imageData = chosenImage.pngData()
```

<br>



Then save it into the CoreData entity like this : 

```swift
let imageData = chosenImage.pngData()

guard let appDelegate = UIApplication.shared.delegate as? AppDelegate else {
  return
}
        
let context = appDelegate.persistentContainer.viewContext
        
guard let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: context) as? User else {
  return
}
        
user.avatar = imageData
        
do {
    try context.save()
} catch {
    print("Could not save. \(error), \(error.localizedDescription)")
}
```

<br>



This will save the image data as [BLOB](https://en.wikipedia.org/wiki/Binary_large_object) in the database (The default database of CoreData is .sqlite).



The .sqlite database is stored in the **/Library/ApplicationSupport** folder, if the app is built and run on the simulator, we can inspect the .sqlite file by navigating Finder to the app's ApplicationSupport directory.

```swift
// path to application support
print(NSSearchPathForDirectoriesInDomains(.applicationSupportDirectory, .userDomainMask, true).last!);

// copy this
```

<br>



In Finder, press <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>G</kbd> to jump to specific folder. Then paste in the path we copied earlier.

![Application Support Path](https://iosimage.s3.amazonaws.com/2019/56-store-image-coredata/appSupport.png)



You will see there's a few files inside the folder, the one we are interested in is the .sqlite file :

![sqlite file](https://iosimage.s3.amazonaws.com/2019/56-store-image-coredata/sqliteFile.png)



I used [DB Browser for SQLite Mac App](https://sqlitebrowser.org) to open the .sqlite file, you can use other .sqlite browser app as well.



Upon inspection, we can see that the avatar column stored a BLOB of binary data of the image:

![binary data](https://iosimage.s3.amazonaws.com/2019/56-store-image-coredata/binaryMode.png)



DB Browser for SQLite has a feature that let us view the image representation of the binary data, select "Image" from the mode and we can view the image :



![imageMode](https://iosimage.s3.amazonaws.com/2019/56-store-image-coredata/imageMode.png)





It's convenient to store image data directly into core data attribute but it can affect your app performance (CoreData Read / Write time) if you have a lot of rows that contain large binary data.



According to [Apple's Core Data Documentation](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/Performance.html#//apple_ref/doc/uid/TP40001075-CH25-SW11): 

> If your application uses Binary Large OBjects (BLOBs) such as image and sound data, you need to take care to minimize overheads. Whether an object is considered small or large depends on an application’s usage. A general rule is that objects smaller than a megabyte are small or medium-sized and those larger than a megabyte are large.



> It is better, however, if you are able to store BLOBs as resources on the file system and to maintain links (such as URLs or paths) to those resources. You can then load a BLOB as and when necessary. 



Apple advise that we store the image file in the File system (eg: the app's /Documents directory) and store the image URL (eg: /Documents/asriel.png)  in the Core Data attributes instead.



It can take quite some effort to implement this, fortunately starting from [iOS 5.0](https://developer.apple.com/library/archive/releasenotes/DataManagement/RN-CoreData/index.html), Apple has provided us the feature '**Allows External Storage**'. To enable this feature, open the core data model, select your binary data attribute, then check the box 'Allows External Storage'.



![allows external storage](https://iosimage.s3.amazonaws.com/2019/56-store-image-coredata/allowsExternal.png)



What this checkbox does is as described below from Apple's documentation:  

> Small data values like image thumbnails may be efficiently stored in a database, but large photos or other media are best handled directly by the file system. You can now specify that the value of a managed object attribute may be stored as an external record—see `setAllowsExternalBinaryDataStorage:`. When enabled, Core Data heuristically decides on a per-value basis if it should save the data directly in the database or store a URI to a separate file which it manages for you. You cannot query based on the contents of a binary data property if you use this option.



In short, after checking this box, CoreData will automatically decide when it will save the image binary data directly to the column, or save the image binary data in the file system and save the path to the image to the column. The decision is largely based on the size of the binary data (if it is quite small, then it will be saved into the column directly).



After checking the box, large image file will be stored separately, and the column stores binary data that reference the path to the image.



The column stores the reference file name to the image (the red square) :

![binary reference path](https://iosimage.s3.amazonaws.com/2019/56-store-image-coredata/binaryPath.png)



In the Application Support folder, press <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>.</kbd> (Period) to reveal hidden file / folder. We will see that theres a hidden **.CoreDataImage_SUPPORT** folder. Open it and we will find another folder named '_EXTERNAL_DATA', the image is stored inside this folder.



![external image data](https://iosimage.s3.amazonaws.com/2019/56-store-image-coredata/imageDataa.png)



<div class="post-subscribe">
    <script async data-uid="444e484e10" src="https://f.convertkit.com/444e484e10/75994f2863.js"></script>
</div>







