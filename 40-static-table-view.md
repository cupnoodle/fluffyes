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





// How to pass table view row tap to view controller

// delegate



// How to pass textfield data in table view row outlet to view controller

// self.children

