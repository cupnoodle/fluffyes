# Help! My app freezes but no error appears

You might have came across situation where suddenly your app hangs and not responding to any user input like this : 



![Frozen](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/frozen.gif)



You might have did some googling before and gotten answer that this is related to **main thread being blocked**. What does "main thread being blocked" mean? 🤔



In the above demo, the code that was executed when the "Freeze!" button is tapped is as follow:

```swift
@IBAction func freezeButtonTapped(_ sender: UIButton) {
    // main thread will be blocked until the whole zip file has 
    // finished downloading
    // meaning UI will not be responsive until it finishes download
    let data = try? Data(contentsOf: URL(string: "https://github.com/fluffyes/AppStoreCard/archive/master.zip")!)
}
```

<br>



`Data(contentsOf: URL)` will attempt to download the data from the URL we provided, as this run on main thread, the main thread is occupied (blocked) with this download task and can't handle our UI interaction hence we feel that the app 'freezes'.



## Core, Thread and Queue

To understand the phrase "main thread is blocked", we need to first understand the concept of thread and concurrency. 



Long time ago, most personal computers have only one CPU core and they can only do one thing/task at a time (ignoring hyperthreading and virtual cores). When you do multiple task on a single-core computer, the CPU splits it time between all these task with very very short interval, giving us mere mortal an illusion that it is running multiple task at once.



![single core](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/singlecore.png)



For a multi-core computer, it can actually perform multiple task at the same time on different core, thus increasing performance.



![multicore](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/multicore.png)



The concept of multi-core can be further extended into multi-threading in an application. Typically we don't explicitly tell which core of a computer to perform a certain task, usually we group a sequence of task (multiple lines of code/ a function etc) into a thread, and then the Operating System (eg: iOS) will decide which core handles which thread, and pass the thread (with task inside) to the core to execute.



![multi thread](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/multithread.png)



By default if we didnt we create any new thread, the code will run on one thread. Say we have a code of two for loops like this : 

```swift
import Foundation

for a in 1...10 {
   print("Default thread printed a - \(a)")
}

for b in 1...10 {
  print("Default thread printed b - \(b)")
}
```

<br>



This will result the following output :

![default thread](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/defaultThread.png)



Code in the same thread is executed sequentially, meaning from top to bottom, the second loop will only be executed after the first loop has finished.







