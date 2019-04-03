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

Just want to jump to the code answer? [Click here](#tldr)

Table of contents: 
1. [Core, Thread and Queue](#core-thread-queue)
2. [Main Thread and Queue](#main)
3. [UI updates should happen in Main thread](#ui-main)
4. [Further Reading](#further)
5. [Playground project demo of queue](#cta)


<span id="core-thread-queue"></span>
## Core, Thread and Queue

To understand the phrase "main thread is blocked", we first need to understand the concept of thread and concurrency. 



Long time ago, most personal computers have only one CPU core and they can only do one thing/task at a time (ignoring hyperthreading and virtual cores). When you do multiple task on a single-core computer, the CPU splits it time between all these task with very very short interval, giving us mere mortal an illusion that it is running multiple task at once.



![single core](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/singlecore.png)



For a multi-core computer, it can actually perform multiple task at the same time on different core, thus increasing performance.



![multicore](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/multicore.png)



The concept of multi-core can be further extended into multi-threading in an application. Typically we don't explicitly tell which core of a computer to perform a certain task, usually we group a sequence of task (multiple lines of code/ a function etc) into a thread, and then the Operating System (eg: iOS) will decide which core handles which thread, and pass the thread (with task inside) to the core to execute.



![multi thread](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/multithread.png)



By default if we didnt we create any new thread, the code will run on one thread (the default thread). Say we have a code of two for loops like this : 

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



![single thread](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/singlethread.png)



Let's change the code above to create a new thread for the first for loop (for loop A) : 

```swift
import Foundation

// create a new thread and execute code inside this new thread
Thread.detachNewThread {
    for a in 1...10 {
        print("New thread printed \(a)")
    }
}

// this is running on default thread
for b in 1...10 {
    print("Default thread printed b - \(b)")
}

```

<br>



If we run this, the output will be like this : 

![concurrency](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/concurrency.png)



The two loops are running at the same time! 😳 This is because the two threads are ran concurrently.



This is what happens under the hood : 

![double thread](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/doublethread.png)



What if your computer has only one core? Most likely your computer core will switch between threads like this : 

![double thread single core](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/doublethreadsinglecore.png)



The core will switch between the two for-loops with a very quick interval (milliseconds) , causing an illusion that both for loops are executed on the same time. Chances are, in a multi-core computer, one of the core might keep switching between different thread/task (actually all of the cores might keep switching between different threads).



[Apple recommends us to migrate away from dealing with thread](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html) directly and use their queue (Dispatch queue / Operation queue) instead. 



Instead of creating a thread to handle task concurrently and managing them, Apple recommends us to put task in a queue and let the operating system to handle the threading stuff like creating threads, placing task into thread, thread communications, etc.



![queue](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/queue.png)


<span id="main"></span>
## Main Thread and Queue

Remember the default thread we mentioned earlier? If we didn't explicitly define a new thread, the code will be executed on the default thread. In iOS, the default thread is also called as **Main thread** or **UI thread**, this is because the user interface (UIKit) of the app runs on the default thread as well, this includes handling button tap, updating label values, scrolling table views, etc..



![main thread](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/mainthread.png)



When we write code inside any UIKit method (eg: viewDidLoad) without explicitly creating a new thread, say like this : 

```swift
import Foundation
import UIKit

class SomeViewController : UIViewController {
  override func viewDidLoad() {
    for a in 1...10 {
       print("Default thread printed a - \(a)")
    }

    for b in 1...10 {
      print("Default thread printed b - \(b)")
    }
  }
}

```

<br>



The two for loops in the code above are running on main thread, making the task on main thread to look like this : 

![mainthread 2](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/mainthread2.png)



The UI is still responsive to our input because the two for loops we added take very tiny amount of time (milliseconds) to run on the main thread. After the loops have finished running, the main thread can respond back to user input.



As you might have guessed, if we run a non-UI task that takes a long time on the main thread, it will cause the UI to be unresponsive as the main thread needs to finish processing the task before it can go back to handling UI (eg: user input such as tap, and also updating the display/graphic on screen).

```swift
@IBAction func freezeButtonTapped(_ sender: UIButton) {
  // download the zip file from Github on main thread
  let data = try? Data(contentsOf: URL(string: "https://github.com/fluffyes/AppStoreCard/archive/master.zip")!)
}
```

<br>

![blocking](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/blocking.png)



In this case, the UI stops responding when the zip file download starts and only starts responding after the zip file download has finished. One of the way to solve this is to move the download zip file task into another thread. As Apple encourage us to use queue instead of managing thread directly, we can wrap the download zip file code with a **global queue** like this :


<span id="tldr"></span>

```swift
@IBAction func freezeButtonTapped(_ sender: UIButton) {
  // send the code to global queue, this will be dispatched to one of the background threads instead of main thread
  DispatchQueue.global().async {
    let data = try? Data(contentsOf: URL(string: "https://github.com/fluffyes/AppStoreCard/archive/master.zip")!)
  }
}
```

<br>



For DispatchQueue, other than creating our own serial / concurrent queue, we can also access the **Main Queue** and **Global Queue** using `DispatchQueue.main()` and `DispatchQueue.global()` . The code placed inside main queue will be passed to the main / UI thread, whereas the code placed inside global queue will be passed to different background threads depending on their priority. 



There's multiple background threads but only one main thread. One thing to note is that main queue is a serial queue, where task will execute following order from top to bottom, whereas global queue is a concurrent queue where multiple tasks might be dispatched to different threads at the same time.

![global and main queue](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/mainglobalqueue.png)



<span id="ui-main"></span>

## UI updates should happen in Main thread

Now that we have moved the zip file download task to background thread using global queue : 

<span id="globalqueue"></span>

```swift
@IBAction func freezeButtonTapped(_ sender: UIButton) {
  // send the code to global queue, this will be dispatched to one of the background threads instead of main thread
  DispatchQueue.global().async {
    let data = try? Data(contentsOf: URL(string: "https://github.com/fluffyes/AppStoreCard/archive/master.zip")!)
  }
}
```

<br>



What if we want to update a label text using the data we have downloaded? If we update any UI element inside a non-main thread, Xcode will show us a runtime warning like this : 

```swift
@IBAction func freezeButtonTapped(_ sender: UIButton) {
  // send the code to global queue, this will be dispatched to one of the background threads instead of main thread
  DispatchQueue.global().async {
    let data = try? Data(contentsOf: URL(string: "https://github.com/fluffyes/AppStoreCard/archive/master.zip")!)
    
    // update UI label, I know its not using the downloaded data
    // its just for demonstration
    self.nameLabel.text = "Zip file downloaded"
  }
}
```

<br><br>

![warning on updating UI in non-main thread](https://iosimage.s3.amazonaws.com/2019/49-help-my-app-freeze/mustBeUsedOnMainThread.png)



[Apple's official documentation](https://developer.apple.com/documentation/code_diagnostics/main_thread_checker) mentioned that updating UI on a thread other than the main thread can result in missed UI updates, visual defects, data corruptions, and crashes.



To fix this issue, we can use wrap the UI element inside main queue (inside the global queue) like this : 

```swift
@IBAction func freezeButtonTapped(_ sender: UIButton) {
  // send the code to global queue, this will be dispatched to one of the background threads
  DispatchQueue.global().async {
    let data = try? Data(contentsOf: URL(string: "https://github.com/fluffyes/AppStoreCard/archive/master.zip")!)
    
    // jump back to main thread to update the UI
    // you can use the data downloaded here
    DispatchQueue.main().async {
      self.nameLabel.text = "Zip file downloaded"
    }
  }
}
```

<br>



One of the key to making performant app is to move as much non-UI related heavy processing to background thread as possible so that user won't experience lag / freeze on the UI.


<span id="further"></span>
## Further Reading

[Dispatch Queues (Concurrency programming guide)](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html)



<span id="cta"></span>

<script async data-uid="644df95883" src="https://f.convertkit.com/644df95883/d3af8370a8.js"></script>

