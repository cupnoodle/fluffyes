# NSURLSession / URLSession tutorial for calling web API



[URLSession](https://developer.apple.com/documentation/foundation/urlsession) (previously NSURLSession) API is introduced in iOS7, to replace the deprecated NSURLConnection.



This post will cover what is URLSession, URLSessionTask and use Swift 4. This post will assume the context of using URLSession for **HTTP** request and getting JSON response in completion block. [Official documentation](https://developer.apple.com/documentation/foundation/urlsession) mentioned that URLSession class natively supports the data, file, ftp, http, and https URL schemes.

Table of contents :
1. [What is URLSession?](#urlsession)
2. [What is URLSessionTask?](#urlsessiontask)
3. [Making GET request using URLSessionDataTask](#get)
4. [Making POST request using URLSessionDataTask](#post)
5. [HTTP App Transport Security issue](#http)
6. [Conclusion](#conclusion)

Want to parse JSON returned from API to object easily using Codable? [Read more about it here](https://fluffy.es/parse-json-using-decodable-protocol/)
<br>
<span id="urlsession"></span>
## What is URLSession?

URLSession is the class responsible for making HTTP request and getting HTTP response. In a really simplified analogy, think of URLSession as a web browser and it can have multiple tabs opening multiple website. Those tabs which request and load website data are URLSessionTask. One URLSession can have multiple URLSessionTask send request to different website.

![URLSession and URLSessionTask](https://iosimage.s3.amazonaws.com/2018/11-urlsession-tutorial/onesession.png)

You can create a instance of URLSession like this :
```swift
// default configuration, store cache and cookie in the disk storage and other urlsession can access it
let config = URLSessionConfiguration.default
let session = URLSession(configuration: config)
```

<br>

If you create multiple instances of URLSession, you can treat it as opening multiple web browsers with multiple tabs, like this : 

![Multiple URLSession](https://iosimage.s3.amazonaws.com/2018/11-urlsession-tutorial/multisession.png)

URLSession instance is initialized using URLSessionConfiguration. [URLSessionConfiguration](https://developer.apple.com/documentation/foundation/urlsessionconfiguration) has three types : 

1. **.default** - The URLSession will save cache / cookie into disk, credentials are saved to keychain
2. **.ephemeral** - This is similar to opening Incognito mode on Chrome or Private browsing on Firefox / Safari, cache / cookie / credential are stored in memory and will be gone once the session is terminated
3. **.background** - This allow the session to perform upload / download task in the background, meaning even if the app is suspended (in background), the upload / download task will still continue.

You can set custom HTTP header, timeout, caching policy, etc in the URLSessionConfiguration object : 

```swift
let config = URLSessionConfiguration.default
config.httpAdditionalHeaders = ["User-Agent":"Legit Safari", "Authorization" : "Bearer key1234567"]
config.timeoutIntervalForRequest = 30
// use saved cache data if exist, else call the web API to retrieve
config.requestCachePolicy = NSURLRequest.CachePolicy.returnCacheDataElseLoad        
```

<br>

You can see different type of cache policy [here in Apple Documentation](https://developer.apple.com/documentation/foundation/nsurlrequest.cachepolicy).

<span id="urlsessiontask"></span>
## What is URLSessionTask?
URLSessionTask is the class responsible for making request to the web API and uploading / downloading data.

There's 3 type of URLSessionTask : 
1. URLSessionDataTask - Use this for sending HTTP GET / POST / PUT / DELETE request, the data retrieved from response is saved into Memory in NSData / Data form
2. URLSessionUploadTask - Use this for uploading file 
3. URLSessionDownloadTask - Use this for downloading file

You can access the response data returned from the URLSession by using completion block or delegate. 

We will be focusing on URLSessionDataTask and access response data using completion block for this post.

<span id="get"></span>
## Making GET request using URLSessionDataTask
We can create a URLSessionDataTask which perform GET request to a web API as below : 

```swift
let config = URLSessionConfiguration.default
let session = URLSession(configuration: config)

let url = URL(string: "https://httpbin.org/anything")!
let task = session.dataTask(with: url) { data, response, error in

    // ensure there is no error for this HTTP response
    guard error == nil else {
        print ("error: \(error!)")
        return
    }
	
    // ensure there is data returned from this HTTP response
    guard let content = data else {	
        print("No data")
        return
    }
	
    // serialise the data / NSData object into Dictionary [String : Any]
    guard let json = (try? JSONSerialization.jsonObject(with: content, options: JSONSerialization.ReadingOptions.mutableContainers)) as? [String: Any] else {
        print("Not containing JSON")
        return
    }
	
    print("gotten json response dictionary is \n \(json)")
    // update UI using the response here
}

// execute the HTTP request
task.resume()

```

<br>

The `session.dataTask(with: url)` method will perform a GET request to the url specified and its completion block (`{ data, response, error in }`) will be executed once response is received from the server.

`JSONSerialization.jsonObject(with: content, options: JSONSerialization.ReadingOptions.mutableContainers) as? [String: Any]` will parse the JSON data returned from web server into a dictionary

`JSONSerialization.ReadingOptions.mutableContainers` option will make the array / dictionary generated from the JSON data be mutable (think it as `var`, means you can modify the data inside the array / dictionary), usually you will want to use this option for parsing JSON data. [More options on the Apple documentation](https://developer.apple.com/documentation/foundation/jsonserialization.readingoptions).

<span id="post"></span>
## Making POST request using URLSessionDataTask
We can create a URLSessionDataTask which perform POST request to a web API as below :

```swift
let config = URLSessionConfiguration.default

let session = URLSession(configuration: config)

let url = URL(string: "https://httpbin.org/anything")!
var urlRequest = URLRequest(url: url)
urlRequest.httpMethod = "POST"

// your post request data
let postDict : [String: Any] = ["name": "axel",
                                "favorite_animal": "fox"]

guard let postData = try? JSONSerialization.data(withJSONObject: postDict, options: []) else {
    return
}

urlRequest.httpBody = postData

let task = session.dataTask(with: urlRequest) { data, response, error in
	
    // ensure there is no error for this HTTP response
    guard error == nil else {
        print ("error: \(error!)")
        return
    }
	
    // ensure there is data returned from this HTTP response
    guard let content = data else {	
        print("No data")
        return
    }
	
    // serialise the data / NSData object into Dictionary [String : Any]
    guard let json = (try? JSONSerialization.jsonObject(with: content, options: JSONSerialization.ReadingOptions.mutableContainers)) as? [String: Any] else {
        print("Not containing JSON")
        return
    }
	
    print("gotten json response dictionary is \n \(json)")
    // update UI using the response here
}

// execute the HTTP request
task.resume()
```

<br>

To perform a web request other than GET, we will need to create a URLRequest object and specify its `httpMethod`, as we are performing POST here, we set it to `urlRequest.httpMethod = "POST"`. You can also set it to PUT , DELETE etc.

We convert the data that will be sent to the server to JSON format using `JSONSerialization.data(withJSONObject: postDict, options: [])` , the options is left blank by using an empty array. You can use the option to make the output json in pretty print format and / or sorted using key. You can read more about this writing option [here on Apple Documentation](https://developer.apple.com/documentation/foundation/jsonserialization.writingoptions).

We then include the converted json data into the httpBody of the URL request.

The `session.dataTask(with: urlRequest)` method will execute the URL request specified and its completion block (`{ data, response, error in }`) will be executed once response is received from the server.

`JSONSerialization.jsonObject(with: content, options: JSONSerialization.ReadingOptions.mutableContainers) as? [String: Any]` will parse the JSON data returned from web server into a dictionary

`JSONSerialization.ReadingOptions.mutableContainers` option will make the array / dictionary generated from the JSON data be mutable (think it as `var`, means you can modify the data inside the array / dictionary), usually you will want to use this option for parsing JSON data. [More options on the Apple documentation](https://developer.apple.com/documentation/foundation/jsonserialization.readingoptions).

<span id="shared"></span>
## Shared URLSession
Similar to `UIApplication.shared` , URLSession has a singleton instance which you can reuse if you are doing simple GET / POST / PUT / DELETE request in your app.

Instead of creating a new session, we can use 
```swift
let taskA = URLSession.shared.dataTask(with: urlRequest) { data, response, error in
...
}

let taskB = URLSession.shared.dataTask(with: urlRequest) { data, response, error in
...
}
```

<br>

As you are reusing the same instance of URLSession for different task, it takes up less memory than using 5 URLSession for different task. Imagine the memory consumption when you open multiple browsers (Firefox , Chrome and Safari) VS only opening one browser. Usually for similar task, I recommend grouping them in the same URLSession, like grouping all JSON call into a session, grouping file downloads call into another session, etc. 

Please don't create a new URLSession for each web request, as Apple Engineer mentioned in the [official forum](https://forums.developer.apple.com/thread/84663) :  
> Creating a session per request is inefficient both on the CPU and, more importantly, on the network.  Specifically, it prevents connection reuse, which can radically slow down back-to-back requests.  This is especially bad for HTTP/2.
>
> We encourage folks to group all similar tasks in a single session, using multiple sessions only if you have different sets of tasks with different requirements (like interactive tasks versus background download tasks).  That means that many simple apps can get away with using a single statically-allocated session.
>
> Quinn “The Eskimo!” 
> Apple Developer Relations, Developer Technical Support, Core OS/Hardware 

<span id="http"></span>
## HTTP App Transport Security issue
If you are making web request to a HTTP endpoint instead of HTTPS, you will see the following error message in the console :  
```
App Transport Security has blocked a cleartext HTTP (http://) resource load since it is insecure. 
Temporary exceptions can be configured via your app's Info.plist file.
```

<br>

It means that Apple's App Transport Security has blocked your web request, well you shouldn't be using HTTP as it is insecure, <s>I can read what HTTP request you sent to server if I am sitting in the same coffee shop as you, [here's how](https://www.howtogeek.com/104278/how-to-use-wireshark-to-capture-filter-and-inspect-packets/) </s>.

If you (or the backend developer) insist on using HTTP, you can bypass the App Transport Security block by adding "Allow Arbitrary Load" key to the Info.plist in your Xcode Project.

Open "Info.plist", click "+" on the top most row ("Information Property List") and type in "App Transport Security Settings".
![App Transport Security Settings](https://iosimage.s3.amazonaws.com/2018/11-urlsession-tutorial/AppTransport.png)

Then click "+" on "App Transport Security Settings" , type in "Allow Arbitrary Loads", then select "YES" for the value.
![Allow Arbitrary Loads](https://iosimage.s3.amazonaws.com/2018/11-urlsession-tutorial/AllowArbitraryLoad.png)

You should be able to make HTTP request call without error now.
<span id="conclusion"></span>
## Conclusion

Using URLSession and URLSessionTask to perform networking function is quite straightforward. For simple web request, using a shared URLSession should suffice. Alamofire library is built on top of URLSession, if you need advanced feature like multipart upload or download, using Alamofire might be easier as you don't need to write much boilerplate code.

<div class="post-subscribe">
  <div class="post-subscribe-left">
    <h4> Get the URLSession Xcode Project File and test it</h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/2018/11-urlsession-tutorial/demoprojectsmall.png"></img>See the code live and tweak it with your own API endpoint
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/67230949/submissions" method="post" data-drip-embedded-form="67230949">
                <div style="margin-bottom: 0.5rem;">
                    <label for="drip-firstname">First Name<span style="color:#952B45;">*</span></label><br />
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
                <span style="font-size: 0.8rem;">Bi-weekly ish iOS Development tips. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>