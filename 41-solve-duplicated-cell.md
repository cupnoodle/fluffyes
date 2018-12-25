# Solving duplicated / repeating cells in Table view



You followed a tutorial to create a simple table view, you managed to create a custom cell with some labels and images, it seems to go well. But as soon as you scroll it, you noticed something is wrong.



When you tap on a cell and scroll it, the cells below get repeated (even though you didn't tap them)! 😱



![repeatingCell](https://iosimage.s3.amazonaws.com/2018/41-solve-duplicated-cell/repeatingCell.gif)

The above example shows a tableview with a to-do task list. When a user tap on the cell, it means the task has been done and a checkmark will be shown. But as user scrolls after tapping a few cells on top, the cells below are also marked as done even though the user didn't tap them before!



Why does this happen? Remember the **dequeueReusableCell(withIdentifier: for:)** method you used inside the cellForRowAtIndexPath method?

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
  
    let cell = tableView.dequeueReusableCell(withIdentifier: todoCellIdentifier, for: indexPath) as! TodoTableViewCell
    
    // assign text to label and stuff
    cell.taskLabel.text = "Task \(indexPath.row)"
    return cell
}
```

<br>



The **dequeue** means that there is a queue used to store table view cells. What does a **queue** mean? 🤔

When you register a cell for a table view like this : 

```swift
self.tableView.register(UINib(nibName: String(describing: TodoTableViewCell.self), bundle: nil), forCellReuseIdentifier: "todoCellIdentifier")
```

<br>

This will create a queue named **todoCellIdentifier**, this queue is used to hold multiple cells of class TodoTableViewCell.

![queue](https://iosimage.s3.amazonaws.com/2018/41-solve-duplicated-cell/newQueue.png)



Here is an illustration showing how the reusable cell queue works : 

![reusable queue](https://iosimage.s3.amazonaws.com/2018/41-solve-duplicated-cell/dequeue.png)



iOS uses a reusable cell queue mechanism to optimize memory usage. Say your tableview has 1000 cells, instead of putting all 1000 cells in memory, a queue is used to store just enough cells to display on the screen (plus a few more cells as buffer), when a new cell is about to appear on screen (due to scrolling), a cell will be taken out of the queue, customized in cellForRowAtIndexPath method, and then inserted to the tableview. 



For the tableview below, there's only ~15 visible cells on the screen at any given time, so the queue will keep around ~18 ish cells in the memory. (This is a lot less than 1000!)



![iPhone SE table](https://iosimage.s3.amazonaws.com/2018/41-solve-duplicated-cell/SETable.png)



When user scrolls down, the top cell that is pushed outside the screen is placed into the top of the queue, and the bottom cell in the queue is taken out and placed to the bottom of the table view in the screen.



Here's how the cell reuse process works : 

![reuse process](https://iosimage.s3.amazonaws.com/2018/41-solve-duplicated-cell/reuseProcess.png)

Notice that we didn't reset the visibility of the checkmark in the **cellForRowAt** method, and cell 18 is actually cell 2 being reused. As we tapped cell 2 before (making its checkmark visible), this property is passed down to cell 18!



To solve this, we can reset the checkmark to make it invisible in **cellForRowAt** : 

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: todoCellIdentifier, for: indexPath) as! TodoTableViewCell
        
    // reset (hide) the checkmark label
    cell.checkLabel.isHidden = true
  
    // reset the task label text
    cell.taskLabel.text = ""
    
    // get the Task object in the taskArray (data source)
    let task = self.taskArray[indexPath.row]
    
    // if the task has marked done, show the checkmark label
    if(task.done){
      cell.checkLabel.isHidden = false
    }
  
    cell.taskLabel.text = "Task \(indexPath.row)"
    return cell
}
```

<br>



To make the **cellForRowAt** method looks cleaner, we can move the resetting code to the **prepareForResue** method in the custom cell class like this : 

```swift
//  TodoTableViewCell.swift
class TodoTableViewCell: UITableViewCell {
    override func prepareForReuse() {
        // reset (hide) the checkmark label
        self.checkLabel.isHidden = true
        
        // reset the task label text
        self.taskLabel.text = ""
    }
}
```

<br>

And in the cellForRowAt method in view controller : 

```swift
// ViewController.swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: todoCellIdentifier, for: indexPath) as! TodoTableViewCell
    
    // get the Task object in the taskArray (data source)
    let task = self.taskArray[indexPath.row]
    
    // if the task has marked done, show the checkmark label
    if(task.done){
      cell.checkLabel.isHidden = false
    }
  
    cell.taskLabel.text = "Task \(indexPath.row)"
    return cell
}
```

<br>



Below is the flow of cell reuse with queue (including prepareForReuse) : 



![prepareReuse](https://iosimage.s3.amazonaws.com/2018/41-solve-duplicated-cell/prepareReuse.png)





Most of the duplicated / repeated cell issues happen because we didn't reset the cell UI elements to hidden / blank state before reusing it in the table view. This can be easily solved by resetting the UI elements data in the **prepareForReuse** function in the custom cell class 🙌.





