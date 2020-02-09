# How to expand and contract height of a UITableView cell when tapped

To avoid confusion, this article focuses on **expanding and contracing the height of a single tableview cell when it is tapped**. I googled around and found out many people used the same terms for inserting new cells under the tapped cell, which is not what this article is about, I will write on this topic in the future.



At the end of this tutorial, you will be able to implement this :

![expanding gif](https://iosimage.s3.amazonaws.com/2020/71-expanding-cell/expanding.gif)





This tutorial assumes you know how to [implement tableview cell with dynamic height](https://fluffy.es/dynamic-height-tableview-cell/) (Automatic Dimension), as in each cell can contain different content height, eg: lengths of label text, different image sizes.



Say we have a table view cell with image and label set like below : 

![expandingCell](https://iosimage.s3.amazonaws.com/2020/71-expanding-cell/constraint.png)



You can download the starter project here : [Expanding cell starter project](https://drive.google.com/file/d/1AaDr1gZauEcpZtdCU9-rSUhgvMZbWDPT/view?usp=sharing)



The image view (avatarImageView) and label (messageLabel) are initialized in the cellForRowAt function : 

```swift
// ViewController.swift

extension VariableViewController : UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 3
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
      
      // if cant dequeue cell, return a default cell
        guard let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath) as? VariableTableViewCell else {
            print("failed to get cell")
            return UITableViewCell()
        }
        
        cell.avatarImageView.image = UIImage(named: avatars[indexPath.row])
        cell.messageLabel.text = messages[indexPath.row]
        cell.messageLabel.numberOfLines = 0
      
        return cell
    }
}
```

<br>



Before we implement the expanding / contracting animation, we would need to declare an IndexSet to keep track of which cell have been expanded, by storing the **row index into the IndexSet**, so that when we tap on the expanded cell, it would contract instead of expanding.



## Use IndexSet to keep track of which cell has been expanded

From [Apple documentation on IndexSet](https://developer.apple.com/documentation/foundation/indexset), 

> A collection of unique integer values that represent the indexes of elements in another collection.



You can think of IndexSet like an array, it can hold multiple integers, but non of the integers are repeating. This way, we won't need to worry about adding repeating row index into the array.



![indexset](https://iosimage.s3.amazonaws.com/2020/71-expanding-cell/indexSet.png)



"Why can't we just use an array of boolean to track which cell has been expanded?" , I heard you, this solution works too, say you can check if a cell has been expanded  like this : 

```swift
// 10 cells
var expandedCells : [Boolean] = Array(repeating: false, count: 10)

if(expandedCells[index] == true)
{
  // ... show the cell expanded
}
```

<br>



This works well if you have a fixed amount of cells in the table view, but if you have dynamic data that will be added from external API, say like Facebook timeline which keep adding more statuses as you scroll, you will need to adjust the size of the expandedCells array as more cells are being added, this process can easily be error prone!



By using IndexSet, we won't need to worry about the size for the dataSource, we just need to add or remove index of the cell that is expanded. 



We can define an IndexSet to keep track of the index of cell expanded in the view controller like this :

```swift
class ViewController: UIViewController {
    
    @IBOutlet weak var tableView: UITableView!
    
    // this will contain the index of the row (integer) that is expanded
    var expandedIndexSet : IndexSet = []
  
    // ...
}
```

<br>



## Animating the cell to expand and contract

Now that we have an IndexSet to keep track of which cell has been expanded, we can layout the cell expanded / contracted state in **cellForRowAt** like this : 



```swift
extension VariableViewController : UITableViewDataSource {
  // ...

  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath) as? VariableTableViewCell else {
      print("failed to get cell")
      return UITableViewCell()
    }

    cell.avatarImageView.image = UIImage(named: avatars[indexPath.row])
    cell.messageLabel.text = messages[indexPath.row]

    // if the cell is expanded
    if expandedIndexSet.contains(indexPath.row) {
      // the label can take as many lines it need to display all text
      cell.messageLabel.numberOfLines = 0
    } else {
      // if the cell is contracted
      // only show first 3 lines
      cell.messageLabel.numberOfLines = 3
    }

    return cell
  }
}
```

<br>



As we want the cell to be expanded / contracted on tap, we then implement the didSelectAtRow delegate function :

```swift
extension ViewController : UITableViewDelegate {
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        
        // if the cell is already expanded, remove it from the indexset to contract it
        if(expandedIndexSet.contains(indexPath.row)){
            expandedIndexSet.remove(indexPath.row)
        } else {
            // if the cell is not expanded, add it to the indexset to expand it
            expandedIndexSet.insert(indexPath.row)
        }
        
        // the animation magic happens here
        // this will call cellForRow for the indexPath you supplied, and animate the changes
        tableView.reloadRows(at: [indexPath], with: .automatic)
    }
}

```

<br>



The key for the animation is calling **tableView.reloadRows(at:, with:)** . You supply an array of rows (indexpath) that needs to be reloaded, then the tableview will reload them by calling **cellForRowAt:** on these cell to layout them again, and animate the transition with the RowAnimation you passed into the **with:** parameter.



![cellForRowAt](https://iosimage.s3.amazonaws.com/2020/71-expanding-cell/cellForRowAt.gif)



Here's a list of available [RowAnimation](https://developer.apple.com/documentation/uikit/uitableview/rowanimation) we can use to animate the cell row changes, but mostly we will use the .automatic or .fade effect as it looks better.



Implementing the cell animation was easier than I originally thought! Thanks **reloadRows** for the heavy lifting.



If you want to check if an UILabel is truncated (with "..." at the end of text because of clipping), I have written a [short extension here](https://gist.github.com/cupnoodle/75fd46d4fd8af309027a3b67dbdd5b74), this might be useful if you are checking whether to show a "read more" button on the tableview cell depending if the label is being truncated. eg. If the the label text is short and didn't get truncated, don't show the "read more" button. 



