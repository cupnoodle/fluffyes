# How to use Static Table View in View Controllers



You want to add a table view to show static content, and then Bam! Xcode shows you this error  

**Static table views are only valid when embedded in UITableViewController instances** , 

What does this mean? How do we solve it? 🤔



![static table view](https://iosimage.s3.amazonaws.com/2018/40-static-table-view/staticTableError.png)



This error seems like an artificial restriction made by Apple, perhaps Apple wants us to separate out the static table view controller to make it look tidier.



As the error mentioned **only valid when embedded in**, we can use a container view to resolve this.

First, remove the static table view from the view controller. Then drag a container view to the view controller.



![Container view](https://iosimage.s3.amazonaws.com/2018/40-static-table-view/containerView.png)



After placing the container view, a new view controller will be created and linked automatically, this is the view controller that controls the contained view. This is also called as **embed view**, which is mentioned in the error message earlier.



As static table view only works on Table view controller, we will proceed to delete the newly created view controller and then drag a Table view controller to the storyboard.

![Embed view controller](https://iosimage.s3.amazonaws.com/2018/40-static-table-view/embedViewController.png)



After placing the Table view controller, select the container view (the small UIView), hold <kbd>control</kbd> and drag it to the Table view controller, then select **Embed**.



![controlDrag](https://iosimage.s3.amazonaws.com/2018/40-static-table-view/controlDrag.png)



![Select embedded](https://iosimage.s3.amazonaws.com/2018/40-static-table-view/chooseEmbed.png)



Change the Table view's **Content** to '**Static cells**'. Now you have a Static table view embed in a view controller! 🙌



## Passing table view row tap to view controller

Most of the time you would want the parent view controller (which have the container view) to perform an action say, segue to another view controller , when a row inside the embedded table view is tapped.



How do we notify the view controller that the embedded table view row is tapped?



There's many way to do this, one of the way to do it is by using [delegate pattern](https://fluffy.es/eli-5-delegate/).



In the table view controller, add a delegate variable which conforms to the **TableViewControllerDelegate** protocol, and the protocol contain some function you want to use to notify the parent view controller.

```swift
class ProfileTableViewController: UITableViewController {
 
    // this would be the parent view controller
    weak var delegate : ProfileTableViewControllerDelegate?
}

protocol ProfileTableViewControllerDelegate {
  func logoutTapped()
}
```

<br>



And in the parent view controller, you can access the table view controller by using `self.children` property. `self.children` will return a list of container view / embedded view controllers from the parent view controller. Since there is only one container view / embedded view controller (the table view controller), we can assure that self.children[0] will be that table view controller. 



```swift
class ViewController: UIViewController {
    
    var tableViewController : ProfileTableViewController?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        
        // .children is a list of container view controller of the parent view controller
        // since you only have one container view,
        // safe to grab the first one ([0]), and cast it to table VC class
        tableViewController = self.children[0] as? ProfileTableViewController
        tableViewController?.delegate = self
    }
}
```

<br>



// How to pass table view row tap to view controller

// delegate


