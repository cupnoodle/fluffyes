## Determine which button got tapped inside UITableViewCell without using tag

Say you have a button inside every cell in a tableview like this : 

![tableview cell](https://iosimage.s3.amazonaws.com/2018/33-uibutton-uitableviewcell-tap/tableView.png)



You want the app to show an alert with message "Subscribed to [youtuber name]" when user tap on the subscribe button. How do you implement this function? You would need to keep track of which cell got tapped and use that index to find the respective youtuber name from an array.


Let's check out the [top result](https://stackoverflow.com/questions/20655060/get-button-click-inside-uitableviewcell) of googling 'button click in uitableviewcell' :

![google](https://iosimage.s3.amazonaws.com/2018/33-uibutton-uitableviewcell-tap/googleResult.png)



The answer was suggesting to use the tag attribute on the button : 

```swift
let youtubers = ["Brian Voong", "Seth Everman", "Dave Lee", "Cybershell", "Bill Wurtz"]

func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
  let cell = tableView.dequeueReusableCell(withIdentifier: cellReuseIdentifier, for: indexPath) as! YoutuberTableViewCell
  cell.youtuberLabel.text = youtubers[indexPath.row]
  
	// assign the index of the youtuber to button tag	
  cell.subscribeButton.tag = indexPath.row
  
  // call the subscribeTapped method when tapped
  cell.subscribeButton.addTarget(self, action: #selector(subscribeTapped(_:)), for: .touchUpInside)
        
    return cell
}

@objc func subscribeTapped(_ sender: UIButton){
  // use the tag of button as index
  let youtuber = youtubers[sender.tag]
  let alert = UIAlertController(title: "Subscribed!", message: "Subscribed to \(youtuber)", preferredStyle: .alert)
  let okAction = UIAlertAction(title: "OK", style: .default, handler: nil)
  alert.addAction(okAction)
        
  self.present(alert, animated: true, completion: nil)
}
```

<br>



This approach works at first glance but I feel like it is a workaround. [Tag property](https://developer.apple.com/documentation/uikit/uiview/1622493-tag) is original intended to use to uniquely identify a view in app (similar to creating an IBOutlet to identify a view), not to use as a data store (in this case, storing the row / index of the item). Abusing tag can quickly lead to a [nightmare](http://doing-it-wrong.mikeweller.com/2012/08/youre-doing-it-wrong-4-uiview.html) like this if you have multiple section :

```swift
cell.likeButton.tag = indexPath.row + 100
...
// I have seen production code which uses tag 101, 102, 103 etc as row in first section; 201, 202, 203 etc as row in second section..
self.tableView.scrollToRow(at: IndexPath(row: sender.tag, section: sender.tag / 100), at: .top, animated: true)
```

<br>

How would I know what is the **/100** is for?! Shouldn't we use something more intuitive than this math calculation?



The stack overflow post has linked to [another post](https://stackoverflow.com/questions/31649220/detect-button-click-in-table-view-ios-xcode-for-multiple-row-and-section) for handling tableviews with multiple sections, and it involve detecting the coordinate of the cell tapped to deduce the index, oh god why 😱.



Passing the index data shouldn't be that difficult! There are multiple ways to go about this, this post will cover using delegate and closure.


## The delegate way

You can read [how delegate works here](https://fluffy.es/eli-5-delegate/) if you are not farmiliar with it yet. To use the delegate way, we will add a **index** integer property (to keep track of the index) and **delegate** property to the cell class.


```swift
//YoutuberTableViewCell.swift

class YoutuberTableViewCell: UITableViewCell {

  @IBOutlet weak var youtuberLabel: UILabel!
    
  @IBOutlet weak var subscribeButton: UIButton!
    
  // used to keep track the index of the cell
  var index : Int = 0
    
  // the delegate
  var delegate : YoutuberTableViewCellDelegate?
    
  override func awakeFromNib() {
    super.awakeFromNib()
    // Initialization code
        
    // Add action to perform when the button is tapped
    self.subscribeButton.addTarget(self, action: #selector(subscribeButtonTapped(_:)), for: .touchUpInside)
  }

  override func setSelected(_ selected: Bool, animated: Bool) {
    super.setSelected(selected, animated: animated)

    // Configure the view for the selected state
  }
    
  @IBAction func subscribeButtonTapped(_ sender: UIButton){
    // ask the delegate (in most case, its the view controller) to 
    // call the function 'subscribeButtonTappedAt' on itself.
    self.delegate?.youtuberTableViewCell(self, subscribeButtonTappedAt: index)
  }
    
}

protocol YoutuberTableViewCellDelegate {
  func youtuberTableViewCell(_ youtuberTableViewCell: YoutuberTableViewCell, subscribeButtonTappedAt index: Int)
}
```

<br>



The **delegate** is any object which conform to the **YoutuberTableViewCellDelegate** protocol, which means the object has to implement the **subscribeButtonTappedAt** function to be able to be the delegate.



In the Tableview data source, we will assign the index and delegate property in **cellForRowAt** function : 

```swift
extension ViewController : UITableViewDataSource {
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: cellReuseIdentifier, for: indexPath) as! YoutuberTableViewCell
    cell.youtuberLabel.text = youtubers[indexPath.row]
    
    // assign the index to cell
    cell.index = indexPath.row
    
    // the 'self' here means the view controller, set view controller as the delegate
    cell.delegate = self
        
    return cell
}
    
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return youtubers.count
  }
}
```

<br>



As the view controller needs to conform to the YoutuberTableViewCellDelegate, we will add the following code : 

```swift
extension ViewController : YoutuberTableViewCellDelegate {
  func youtuberTableViewCell(_ youtuberTableViewCell: YoutuberTableViewCell, subscribeButtonTappedAt index: Int) {
    // use the passed index to select the correct youtuber
    let youtuber = youtubers[index]
    
    // show alert
    let alert = UIAlertController(title: "Subscribed!", message: "Subscribed to \(youtuber)", preferredStyle: .alert)
    let okAction = UIAlertAction(title: "OK", style: .default, handler: nil)
    alert.addAction(okAction)
    
    self.present(alert, animated: true, completion: nil)
  }
}
```

<br>

The code above will be executed when user tap on the subscribe button on each cell, and the index will be passed via this function.



## The closure way

use closure or delegate 