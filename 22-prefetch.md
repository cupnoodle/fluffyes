# Smoothen your table view data loading using UITableViewDataSource Prefetching

Say you have a view controller with a table view, and you want to load 100 rows of data into it, these data are retrieved from a web API, how would you approach this?

This post assume that you already knew what is [URLSession](https://fluffy.es/nsurlsession-urlsession-tutorial/), [JSON parsing](https://fluffy.es/parse-json-using-decodable-protocol/), [delegate](https://fluffy.es/eli-5-delegate/) and tableview.

![tableView Demo](https://iosimage.s3.amazonaws.com/2018/22-prefetch/tableViewSample.png)

Table of contents :
1. [Load all data at once](#loadall)
2. [Load data by batch when user reach the bottom](#batch)
3. [Introducing UITableViewDataSource Prefetching](#intro)
4. [How prefetching works](#how)
5. [Example - Just in time data loading using Prefetch](#example)
6. [Summary](#summary)

<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> Download the sample project to follow along the post</h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/xproj.png" style="max-width: 150px;"></img>Get sample Xcode project containing code of  Prefetch loading, Batch loading and load all at once.
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/875465538/submissions" method="post" data-drip-embedded-form="875465538">
                <div style="margin-bottom: 0.5rem;">
                    <label for="drip-firstname">Name<span style="color:#952B45;">*</span></label><br />
                    <input type="text" id="drip-firstname" name="fields[firstname]" value="" />
                </div>
                <div>
                    <label for="drip-email">Email Address<span style="color:#952B45;">*</span></label><br />
                    <input type="email" id="drip-email" name="fields[email]" value="" />
                </div>
              <div>
                <br>
                <input type="submit" value="Send me the project file!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips to help you become a better iOS developer.<br> No Spam. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>

<span id="loadall"></span>
## 1 - Load all 100 data at once
The simplest approach would be loading all 100 data using URLSession at once in `viewDidLoad`, then update the table view data source and reload table view after the response is retrieved.

```swift
// Load all data (100 rows) at once
URLSession(configuration: URLSessionConfiguration.default).dataTask(with: URL(string: "https://jsonplaceholder.typicode.com/posts")!) { data, response, error in
  // ensure there is data returned from this HTTP response
  guard let data = data else {
    print("No data")
    return
  }
  
  // Parse JSON into Post array struct using JSONDecoder
  guard let posts = try? JSONDecoder().decode([Post].self, from: data) else {
    print("Error: Couldn't decode data into post model")
    return
  }
  
  // postArray of data source
  self.postArray = posts
  
  // Make sure to update UI in main thread
  DispatchQueue.main.async {
    self.postTableView.reloadData()
  }
}.resume()
```

<br>

This approach is straightforward but getting 100 rows at once and loading them to cells might take longer time / a spike in memory usage. And in some case it is impossible to get all the data in one go (eg: Facebook timeline, too much info to fit into iPhone memory if the facebook app fetched all of your statuses since creation of your account)

<span id="batch"></span>
## 2 - Load data by batch when user reaches bottom
A more advanced approach would be loading data by batches, like loading the initial 20 rows, then load another 20 when user has reached the bottom, this is similar to how Facebook / Twitter load their timelines.

<video width="320" height="578" controls loop>
    <source src="https://iosimage.s3.amazonaws.com/2018/22-prefetch/batch.mp4" type="video/mp4">
</video>

<br>


The batch loading code might be similar to this : 
```swift
class BatchViewController: UIViewController, UITableViewDataSource{

  @IBOutlet weak var coinTableView: UITableView!
  
  var coinArray : [Coin] = []
  
  let baseURL = "https://api.coinmarketcap.com/v2/ticker/?"
  
  // fetch 15 items for each batch
  let itemsPerBatch = 15
  
  // current row from database
  var currentRow : Int = 1
  
  // URL computed by current row
  var url : URL {
    return URL(string: "\(baseURL)start=\(currentRow)&limit=\(itemsPerBatch)")!
  }
  
  // ... skipped viewDidLoad stuff
    
  // MARK : - Tableview data source
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    // +1 to show the loading cell at the last row
    return self.coinArray.count + 1
  }
  
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    // if reached last row, load next batch
    if indexPath.row == self.coinArray.count {
      let cell = tableView.dequeueReusableCell(withIdentifier: loadingCellIdentifier, for: indexPath) as! LoadingTableViewCell
      loadNextBatch()
      return cell
    }

    // else show the cell as usual
    let cell = tableView.dequeueReusableCell(withIdentifier: coinCellIdentifier , for: indexPath) as! CoinTableViewCell
    
    // get the corresponding post object to show from the array
    let coin = coinArray[indexPath.row]
    cell.configureCell(with: coin)
    
    return cell
  }
  
  // MARK : - Batch
  func loadNextBatch() {
    URLSession(configuration: URLSessionConfiguration.default).dataTask(with: url) { data, response, error in
      
      // Parse JSON into array of Car struct using JSONDecoder
      guard let coinList = try? JSONDecoder().decode(CoinList.self, from: data!) else {
        print("Error: Couldn't decode data into coin list")
        return
      }
      
      // contain array of tuples, ie. [(key : ID, value : Coin)]
      let coinTupleArray = coinList.data.sorted {$0.value.rank < $1.value.rank}
      for coinTuple in coinTupleArray {
        self.coinArray.append(coinTuple.value)
      }
      
      // increment current row
      self.currentRow += self.itemsPerBatch
      
      // Make sure to update UI in main thread
      DispatchQueue.main.async {
        self.coinTableView.reloadData()
      }
      
    }.resume()
  }

}
```

<br>

This approach works reasonably well and it lessen the burden on memory as it only load 15 items per batch and only load it when user reached the bottom. One minor inconvenience is that user might need to wait for the next batch to finish load when they reached the bottom. What if we can make it so that the upcoming rows are loaded just before they are displayed on screen? Wouldn't it be better that the user doesn't have to see the loading screen at all? 🤔

<span id="intro"></span>
## Introducing UITableViewDataSource Prefetching

**UITableViewDataSourcePrefetching** protocol for UITableView and UICollectionView is **available since iOS 10**. The delegate function `tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath])` will be called when a row is near the display area, you can call the web API and load them inside this delegate function.

![Prefetch rows](https://iosimage.s3.amazonaws.com/2018/22-prefetch/prefetchRows.png)

There's two function in the `UITableViewDataSourcePrefetching` protocol :

```swift
protocol UITableViewDataSourcePrefetching {
  // This is called when the rows are near to the visible area
  public func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath])
  
  // This is called when the rows move further away from visible area, eg: user scroll in an opposite direction
  optional public func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath])
}
```
<br>

Similar to `tableView.dataSource` from `UITableViewDataSource`, there's another property for table view to store the Prefetch Data Source, `prefetchDataSource`.

```swift
// similar to tableview.dataSource = self
tableView.prefetchDataSource = self
```

<br>
<span id="how"></span>

## How prefetching (kinda) works
Here are just my observations as Apple didn't mention exactly how many rows outside of visible area will be prefetched.

1. `tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) ` doesn’t get called for rows which are visible on screen initially without scrolling.
2. Right after initial visible rows became visible, `tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath])` will be called, and the `indexPaths` variable contain around 10 rows nearest to the visible area.
3. Depending on the scrolling speed, the `indexPaths` in prefetch method will contain different amount of rows. On normal scrolling speed, usually it will contain 1 row. If you scroll at a fast speed, it will contain multiple rows.

Below is a demo video of calling the `prefetchRowsAt` and `cancelPrefetchingForRowsAt` method from  `UITableViewDataSourcePrefetching`.

```swift
func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
		
  print("prefetching row of \(indexPaths)")
}
	
func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath]) {
		
  print("cancel prefetch row of \(indexPaths)")
}
```

<br>

<video width="658" height="436" controls loop>
    <source src="https://iosimage.s3.amazonaws.com/2018/22-prefetch/fetchDemo.mp4" type="video/mp4">
</video>

<br>

<span id="example"></span>
## Example - Just in time data loading using Prefetch

For this example, we will load the top 100 stories from [Hacker News API](https://github.com/HackerNews/API) using Prefetch. Assume we already got the news ID from the [Top Stories API](https://hacker-news.firebaseio.com/v0/topstories.json?print=pretty) . We will display the news using the newsID as ordered below: 

```swift
let newsID = [17549050, 17549099, 17548768, 17546915, 17534858, ... ]
```

<br>

In the view controller, we will use an array to store News object retrieved from the API, this will be the datasource for the tableview. We also have another array to keep track of the URLSessionDataTask used to call the API.

```swift
class PrefetchViewController: UIViewController {	
  // this is the data source for table view
  // fill the array with 100 nil first, then replace the nil with the loaded news object at its corresponding index/position
  var newsArray : [News?] = [News?](repeating: nil, count: 100)
	
  // store task (calling API) of getting each news
  var dataTasks : [URLSessionDataTask] = []
}
```

<br>

In `viewDidLoad()` , we set up the table view's data source and prefetch data source to the view controller itself.

```swift
override func viewDidLoad() {
  super.viewDidLoad()
  newsTableView.dataSource = self
  newsTableView.prefetchDataSource = self
}
```

<br>

Here is the function to fetch the news data from the API. We will fetch the data by creating a URLSessionDataTask, and after this data task is started, we will append it to the `dataTasks` array we defined earlier.

```swift
// the 'index' parameter indicates the row index of tableview
// we will fetch and show the correspond news data for that row
func fetchNews(ofIndex index: Int) {
  let newsID = newsIDs[index]
  let url = URL(string: "https://hacker-news.firebaseio.com/v0/item/\(newsID).json")!
  
  // if there is already an existing data task for that specific news url, it means we already loaded it previously / currently loading it
  // stop re-downloading it by returning this function
  if dataTasks.index(where: { task in
    task.originalRequest?.url == url
  }) != nil {
    return
  }
  
  let dataTask = URLSession.shared.dataTask(with: url) { data, response, error in
    guard let data = data else {
      print("No data")
      return
    }
    
    // Parse JSON into array of Car struct using JSONDecoder
    guard let news = try? JSONDecoder().decode(News.self, from: data) else {
      print("Error: Couldn't decode data into news")
      return
    }
    
    // replace the initial 'nil' value with the loaded news
    // to indicate that the news have been loaded for the table view
    self.newsArray[index] = news
    
    // Update UI on main thread
    DispatchQueue.main.async {
      let indexPath = IndexPath(row: index, section: 0)
      // check if the row of news which we are calling API to retrieve is in the visible rows area in screen
      // the 'indexPathsForVisibleRows?' is because indexPathsForVisibleRows might return nil when there is no rows in visible area/screen
      // if the indexPathsForVisibleRows is nil, '?? false' will make it become false
      if self.newsTableView.indexPathsForVisibleRows?.contains(indexPath) ?? false {
        // if the row is visible (means it is currently empty on screen, refresh it with the loaded data with fade animation
        self.newsTableView.reloadRows(at: [IndexPath(row: index, section: 0)], with: .fade)
      }
    }
  }
  
  // run the task of fetching news, and append it to the dataTasks array
  dataTask.resume()
  dataTasks.append(dataTask)
}
```

<br>

As a refresh from [ELI 5: Optional](https://fluffy.es/eli-5-optional/#optional-chaining), "self.newsTableView .indexPathsForVisibleRows? .contains(indexPath)" contains optional chaining, as `.indexPathsForVisibleRows?` might return nil when there are no visible rows (no row at all) in the table view, if it return nil, the whole statement became nil. The `?? false` at the back is [nil coalescing](https://fluffy.es/eli-5-optional/#nil-coalescing), it means that if "indexPathsForVisibleRows? .contains(indexPath)" returns nil, the value after the double quotation mark will be used, which is `false`.

After the particular news data is loaded, if the row that is supposed to show the news data is in the visible rows area (it is currently blank now), we will update / reload that row with the loaded news data.

To optimize network calls, we are going to add a `cancelFetchNews` function which will cancel the dataTask used to fetch a specific news when user scroll away from the row that is supposed to show that news. This function will be used in the `cancelPrefetchingForRowsAt` delegate method.

```swift
// the 'index' parameter indicates the row index of tableview
func cancelFetchNews(ofIndex index: Int) {
  let newsID = newsIDs[index]
  let url = URL(string: "https://hacker-news.firebaseio.com/v0/item/\(newsID).json")!
  
  // get the index of the dataTask which load this specific news
  // if there is no existing data task for the specific news, no need to cancel it
  guard let dataTaskIndex = dataTasks.index(where: { task in
    task.originalRequest?.url == url
  }) else {
    return
  }
  
  let dataTask =  dataTasks[dataTaskIndex]
  
  // cancel and remove the dataTask from the dataTasks array
  // so that a new datatask will be created and used to load news next time
  // since we already cancelled it before it has finished loading
  dataTask.cancel()
  dataTasks.remove(at: dataTaskIndex)
}
```

<br>

Now we can plug the `fetchNews` function into the `cellForRowAtIndexPath` method, if the cell on screen didnt have its corresponding news data loaded, we will fetch the news data from API.

```swift
extension PrefetchViewController : UITableViewDataSource {
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return newsArray.count
  }
  
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: newsCellIdentifier, for: indexPath) as! NewsTableViewCell
    
    // get the corresponding news object to show from the array
    if let news = newsArray[indexPath.row] {
      cell.configureCell(with: news)
    } else {
      // if the news havent loaded (nil havent got replaced), reset all the label
      cell.truncateCell()
      
      // fetch the news from API
      self.fetchNews(ofIndex: indexPath.row)
    }
    
    return cell
  }
}
```

<br>

`configureCell` is a custom method I wrote to assign the News data to different labels on the cell. `truncateCell` just set the labels text to blank.

Here is the secret sauce of this post, we will call the `fetchNews` function to retrieve the news data for the rows that are near to visible rows area , but still not visible yet. So that user can have the illusion that there is no loading delay 👀 (data is already loaded before the row is visible on screen, shhhh).

```swift
extension PrefetchViewController : UITableViewDataSourcePrefetching {
  func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {

    // fetch News from API for those rows that are being prefetched (near to visible area)
    for indexPath in indexPaths {
      self.fetchNews(ofIndex: indexPath.row)
    }
  }
  
  func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath]) { 
   
    // cancel the task of fetching news from API when user scroll away from them
    for indexPath in indexPaths {
      self.cancelFetchNews(ofIndex: indexPath.row)
    }
  }
}
```

<br>

We will also call `cancelFetchNews` in the delegate method `cancelPrefetchingForRowsAt` when user scroll away from the loading row. This way, we can optimize the network call by cancelling it since we dont need it at the moment.

The end result will look like below. Aside from the initial loading, user won't notice there's loading going on (hopefully 👀), making it seems smooth : 

<video width="318" height="566" controls loop>
    <source src="https://iosimage.s3.amazonaws.com/2018/22-prefetch/smoothtrim.mp4" type="video/mp4">
</video>

<br>

<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> Try out Prefetching yourself!</h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/xproj.png" style="max-width: 150px;"></img>Get sample Xcode project containing Prefetch loading, Batch loading and load all at once, try them out!
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/875465538/submissions" method="post" data-drip-embedded-form="875465538">
                <div style="margin-bottom: 0.5rem;">
                    <label for="drip-firstname">Name<span style="color:#952B45;">*</span></label><br />
                    <input type="text" id="drip-firstname" name="fields[firstname]" value="" />
                </div>
                <div>
                    <label for="drip-email">Email Address<span style="color:#952B45;">*</span></label><br />
                    <input type="email" id="drip-email" name="fields[email]" value="" />
                </div>
              <div>
                <br>
                <input type="submit" value="Send me the project file!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips.<br> Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>

<span id="summary"></span>
## Summary
The prefetch method provided by Apple allows us to lazy load data, as in load it just before the row is presented, this allow for a better user experience as user will not see the loading process aside from the initial one (hopefully they dont scroll like crazy, of course).

Granted, sometimes it might be not feasible to use this prefetch due to HTTP API limitations set by backend developer / company. Feel free to use whichever method that you think will best deliver the user experience on tableview!

[Apple's documentation on UITableViewDataSourcePrefetching](https://developer.apple.com/documentation/uikit/uitableviewdatasourceprefetching)