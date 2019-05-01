# Download files sequentially using URLSession inside OperationQueue

You might have tried to download files sequentially using operation queue like this :



```swift
let queue = OperationQueue()
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