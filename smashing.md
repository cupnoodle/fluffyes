# 5 iOS Performance Tricks to make your app feel more performant



## 1. Dequeue reusable cell

When you search for tutorials on implementing table view on the internet, most of them will instruct you to use the **tableView.dequeueReusableCell(withIdentifier:, for:)** method in **tableView(_:cellForRowAt:)** to reuse a cell. What is the function of reusable cell? To explain this, we will first look at scenario without using reusable cell.



Say you have a table view with 1000 rows, without using reusable cells, we will have to create a new cell for each row like this : 

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    // create a new cell whenever cellForRowAt is called
    let cell = UITableViewCell()
    cell.textLabel?.text = "Cell \(indexPath.row)"
    return cell
}
```



As you might have thought, this will add 1000 cells into your iPhone memory as your scroll to the bottom. Imagine each cell contain an UIImageView and lots of text, loading them all at once might cause the app to run out of memory!



To resolve this, Apple provided us the **dequeueReusableCell(withIdentifier: , for:)** method. Cell reusing works by placing the cell that is no longer visible in the screen into a queue, and when a new cell is going to be visible in the screen (say, the bottom upcoming cell when user scrolls), the table view will retrieve a cell from this queue and do modification in **cellForRowAt indexPath:** method.



![dequeue](smashing/dequeue.png)



By using a queue to store cells, the tableview doesn't need to create 1000 cells. Instead, it only need just enough cells to cover the area of table view.



By using dequeueReusableCell, we can reduce the memory used by the app and make it less prone to running out of memory!



## 2. Using Launch Images that look like the initial screen

