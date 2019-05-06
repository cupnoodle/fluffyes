# Download files sequentially using URLSession inside OperationQueue

There might be scenario where your app have to download external resource files from the internet, as URLSession is asynchronous, if we call multiple dataTask or downloadTask in the code, they might all run together and make multiple requests to the server at the same time. 



```swift
// yes I know using force unwrap is bad
// urls are ordered in alphabetical order
let urls = [
    URL(string: "https://github.com/fluffyes/AppStoreCard/archive/master.zip")!,
    URL(string: "https://github.com/fluffyes/currentLocation/archive/master.zip")!,
    URL(string: "https://github.com/fluffyes/DispatchQueue/archive/master.zip")!,
    URL(string: "https://github.com/fluffyes/dynamicFont/archive/master.zip")!,
    URL(string: "https://github.com/fluffyes/telegrammy/archive/master.zip")!
]

// call multiple urlsession downloadtask, these will all run at the same time!
for url in urls {
    print("start fetching \(url.absoluteString)")
    URLSession.shared.downloadTask(with: url, completionHandler: { (tempURL, response, error) in
        print("finished fetching \(url.absoluteString)")
    }).resume()
}
```

<br>



Some external API might have restriction that only 1 request can be made at the same time from the same IP address, performing multiple requests at the same time might result in HTTP status 403 Forbidden ! Or worse, all the heavy downloadTasks run concurrently and your app gets overloaded and crash!



You might have heard of **OperationQueue** and tried to download files sequentially using operation queue like this :



```swift
let queue = OperationQueue()
// set maxConcurrentOperationCount to 1 so that only one operation can be run at one time
queue.maxConcurrentOperationCount = 1

// yes I know using force unwrap is bad
// urls are ordered in alphabetical order
let urls = [
  URL(string: "https://github.com/fluffyes/AppStoreCard/archive/master.zip")!,
  URL(string: "https://github.com/fluffyes/currentLocation/archive/master.zip")!,
  URL(string: "https://github.com/fluffyes/DispatchQueue/archive/master.zip")!,
  URL(string: "https://github.com/fluffyes/dynamicFont/archive/master.zip")!,
  URL(string: "https://github.com/fluffyes/telegrammy/archive/master.zip")!
]

for url in urls {
    
  let operation = BlockOperation(block: {
    print("start fetching \(url.absoluteString)")
    URLSession.shared.downloadTask(with: url, completionHandler: { (tempURL, response, error) in
      print("finished fetching \(url.absoluteString)")
    }).resume()
  })

  queue.addOperation(operation)
}
```

<br>

 

But when you run it, all the download tasks are being executed at the same time even though **maxConcurrentOperationCount** is set to 1 ! 😱

![parallelWtf](https://iosimage.s3.amazonaws.com/2019/53-download-files-sequentially/parallelwtf.gif)



Why did this happen? 



This is because of URLSession dataTask / downloadTask  **.resume()** method is asynchronous, meaning it will return / complete immediately and move to the next line before the response is retrieved.



Say we have the following code : 

```swift
print("before URLSession downloadTask")
URLSession.shared.downloadTask(with: url, completionHandler: { (tempURL, response, error) in
    print("finished downloadTask")
}).resume()
print("after URLSession downloadTask")
```

<br>



This will result on the following output : 

```
before URLSession downloadTask
after URLSession downloadTask
finished downloadTask
```

<br>



Your program will continue to execute the line **print("after URLSession downloadTask")** right after calling **.resume()** without waiting for the response to be retrieved.



Operation has 3 states (ready, executing, finish), the way block operation works is that right before the code inside the block is executed, it is in **ready** state, when the code inside the block is being executed, it is in **executed** state, and when the last line of the code inside the block has finished executing, its state become **finished** , and the operation queue will move on to another operation.



![Ready executing finish](https://iosimage.s3.amazonaws.com/2019/53-download-files-sequentially/readyFinishedLine.png)



As you have guessed, the operation state will become finished right after it execute the **print("after URLSession downloadTask")** line, even before the downloadTask has completed, and then the operation queue will move on to another block operation and repeat the same thing. Before the first downloadTask is completed, all other downloadTasks has been started (resumed) !



This is why all the download tasks are being executed at (almost) the same time even though **maxConcurrentOperationCount** is set to 1.



As BlockOperation will execute code from top to bottom and set the state to finish when the last line has been executed, we can't use BlockOperation to make sequential URLSession calls, we will need to create our own subclass of **Operation**. In this custom subclass, we will define when the operation is finished, which is **when the downloadTask has been completed**, instead of when it reach the last line of code.



## Custom Operation Subclass

Let's create a custom operation subclass named **DownloadOperation** :

```swift
class DownloadOperation : Operation {
  
  // declare the download task as variable
  // so we can call self.task.resume() to start the download
  // and also call self.task.cancel() to cancel the download if needed
  // assume the task will never be nil since it will be created during init()
  
  private var task : URLSessionDownloadTask!
}
```

<br>



Operation class has three boolean variables indicating the status of the operation :

1. isReady
2. isExecuting
3. isFinished



We will need to override the value of these three variables as we want to handle the state of the operation manually (instead of following the default ready -> executing -> finish state mentioned previously).



To ease the management of state, we will create a custom enum **OperationState** with three possible value : ready, executing and finished. Next, we create another variable **state** with that enum type to keep track of the operation state. Then we override the Operation class variables **isReady** , **isExecuting** and **isFinished** .



```swift
class DownloadOperation : Operation {
    
    private var task : URLSessionDownloadTask!
    
    enum OperationState : Int {
        case ready
        case executing
        case finished
    }
    
    // default state is ready (when the operation is created)
    private var state : OperationState = .ready {
        willSet {
            self.willChangeValue(forKey: "isExecuting")
            self.willChangeValue(forKey: "isFinished")
        }
        
        didSet {
            self.didChangeValue(forKey: "isExecuting")
            self.didChangeValue(forKey: "isFinished")
        }
    }
    
    override var isReady: Bool { return state == .ready }
    override var isExecuting: Bool { return state == .executing }
    override var isFinished: Bool { return state == .finished }
}
```

<br>



You might notice that there is additional code inside the **state** variable declaration, the **willChangeValue()** and **didChangeValue()** methods inside the willSet and didSet blocks. The code inside **willSet** block will be called right before the variable value is updated (eg: `state = .finished`). The code inside **didSet** block will be called right after the variable value is updated.



The way OperationQueue works is that the queue will observe the value of **isExecuting** and **isFinished** variables of the operation, the queue will move on to process the next operation once the current operation notifies the queue that it has finished executing. The **willChangeValue** and **didChangeValue** methods are used to notify the queue that the value of the variable (the forKey refers to the variable name) has changed, once the queue receive this notification, the queue will check for the value of **isExecuting** and **isFinished** variables. If the value of **isFinished** is true, then the queue will move on to the next operation.



![key value observing graph](https://iosimage.s3.amazonaws.com/2019/53-download-files-sequentially/kvo.png)



This pattern of value observing and notification is known as Key-Value Observing (KVO), you can read more about it on Apple's [Key-Value Observing Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOCompliance.html)



Next, we will add a custom init method for the **DownloadOperation** class to allow us to pass an existing URLSession, URL  and an optional custom completion handler (the function you want to run after the download has completed) into it : 

```swift
class DownloadOperation : Operation {
    
    // ...
		
    init(session: URLSession, downloadTaskURL: URL, completionHandler: ((URL?, URLResponse?, Error?) -> Void)?) {
        super.init()
        
        // use weak self to prevent retain cycle
        task = session.downloadTask(with: downloadTaskURL, completionHandler: { [weak self] (localURL, response, error) in
            
            /* 
            if there is a custom completionHandler defined, 
            pass the result gotten in downloadTask's completionHandler to the 
            custom completionHandler
            */
            if let completionHandler = completionHandler {
                // localURL is the temporary URL the downloaded file is located
                completionHandler(localURL, response, error)
            }
            
           /* 
             set the operation state to finished once 
             the download task is completed or have error
           */
            self?.state = .finished
        })
    }
}
```

<br>



And finally we need to override the **start()** and **cancel()** methods of the operation, start() contains code that will be run when the queue perform the operation, and cancel() contains code that will be run when the operation gets cancelled while in the mid of executing.

```swift
class DownloadOperation : Operation {
  
    // ...
  
    override func start() {
      /* 
      if the operation or queue got cancelled even 
      before the operation has started, set the 
      operation state to finished and return
      */
      if(self.isCancelled) {
          state = .finished
          return
      }
      
      // set the state to executing
      state = .executing
      
      print("downloading \((self.task.originalRequest?.url?.absoluteString)")
            
      // start the downloading
      self.task.resume()
  }

  override func cancel() {
      super.cancel()
    
      // cancel the downloading
      self.task.cancel()
  }
}
```

<br>



According to [Apple's documentation](https://developer.apple.com/documentation/foundation/nsoperation?language=objc#1661262), you should never call **super.start()** inside the overrided start() function : 

> At no time in your `start` method should you ever call `super`.



and also that we need to check if the operation itself was cancelled before start :

> Your `start` method should also check to see if the operation itself was cancelled before actually starting the task. 



Combining all the code together, we will get the class like this : 

```swift
class DownloadOperation : Operation {
    
    private var task : URLSessionDownloadTask!
    
    enum OperationState : Int {
        case ready
        case executing
        case finished
    }
    
    // default state is ready (when the operation is created)
    private var state : OperationState = .ready {
        willSet {
            self.willChangeValue(forKey: "isExecuting")
            self.willChangeValue(forKey: "isFinished")
        }
        
        didSet {
            self.didChangeValue(forKey: "isExecuting")
            self.didChangeValue(forKey: "isFinished")
        }
    }
    
    override var isReady: Bool { return state == .ready }
    override var isExecuting: Bool { return state == .executing }
    override var isFinished: Bool { return state == .finished }
  
    init(session: URLSession, downloadTaskURL: URL, completionHandler: ((URL?, URLResponse?, Error?) -> Void)?) {
        super.init()
        
        // use weak self to prevent retain cycle
        task = session.downloadTask(with: downloadTaskURL, completionHandler: { [weak self] (localURL, response, error) in
            
            /* 
            if there is a custom completionHandler defined, 
            pass the result gotten in downloadTask's completionHandler to the 
            custom completionHandler
            */
            if let completionHandler = completionHandler {
                // localURL is the temporary URL the downloaded file is located
                completionHandler(localURL, response, error)
            }
            
           /* 
             set the operation state to finished once 
             the download task is completed or have error
           */
            self?.state = .finished
        })
    }

    override func start() {
      /* 
      if the operation or queue got cancelled even 
      before the operation has started, set the 
      operation state to finished and return
      */
      if(self.isCancelled) {
          state = .finished
          return
      }
      
      // set the state to executing
      state = .executing
      
      print("downloading \((self.task.originalRequest?.url?.absoluteString)")
            
      // start the downloading
      self.task.resume()
  }

  override func cancel() {
      super.cancel()
    
      // cancel the downloading
      self.task.cancel()
  }
}
```

<br>



