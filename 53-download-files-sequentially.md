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

// call multiple urlsession downloadtask, this all will run at the same time!
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
    print("finished fetching \(url.absoluteString)")
}).resume()
print("after URLSession downloadTask")
```

<br>



This will result on the following output : 

```
before URLSession downloadTask
after URLSession downloadTask
finished fetching https://github.com/fluffyes/AppStoreCard/archive/master.zip
```

<br>



Your program will continue to execute the `print("after URLSession downloadTask")` right after calling **.resume()** without waiting the response to be retrieved.



The way that block operation in queue works is that 

// explain code in block operation, finish state