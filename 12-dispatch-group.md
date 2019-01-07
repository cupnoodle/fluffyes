# Perform function (UI update etc) only after all the network requests have completed using Dispatch Group



Let say you have two different network call function, one to get list of users and one to get list of avatars :

```swift
// Get List of Users
Alamofire.request("https://httpbin.org/get?users").validate().responseJSON { response in
    switch response.result {
    case .success:
        print("Get User Successful")
        // put user into users array
		...
		
    case .failure(let error):
        print(error)
    }
}

// Get List of Avatars
Alamofire.request("https://httpbin.org/get?avatars").validate().responseJSON { response in
    switch response.result {
    case .success:
        print("Get Avatars Successful")
        // put avatar into avatars array
		...
		
    case .failure(let error):
        print(error)
    }
}
```

<br>

And you want to update the tableview showing users and avatars only after **BOTH** network call has finished, how do you do it?

Placing `tableView.reloadData()` in completion handler of either network call doesn't work as the other network call might not finish when the current one finish, making the tableView looks incomplete :

```swift
// Get List of Users
Alamofire.request("https://httpbin.org/get?users").validate().responseJSON { response in
    switch response.result {
    case .success:
        print("Get User Successful")
        // put user into users array
		...
        tableView.reloadData()
        // what if get avatar request havent returned yet? 
        // the table view would look empty without avatar
		
    case .failure(let error):
        print(error)
    }
}
```

<br>


How do you perform `tableView.reloadData()` only after **BOTH** http request is finished?

Answer : use a **Dispatch Group**

Apple introduced [Dispatch Group](https://developer.apple.com/documentation/dispatch/dispatchgroup) in iOS 7.0 to allow us to track when multiple asynchronous tasks have completed.

We can use dispatch group to perform a function after both network request has completed like this :  

```swift
// Create a dispatch group
let userListDispatchGroup = DispatchGroup()

// Get List of Users
userListDispatchGroup.enter()
Alamofire.request("https://httpbin.org/get?users").validate().responseJSON { response in

    switch response.result {
    case .success:
        print("Get User Successful")
        // put user into users array
		...
		
    case .failure(let error):
        print(error)
    }
    
    // leave the dispatch group after the network request is complete and data is parsed
    userListDispatchGroup.leave()
}

// Get List of Avatars
userListDispatchGroup.enter()
Alamofire.request("https://httpbin.org/get?avatars").validate().responseJSON { response in
	
    switch response.result {
    case .success:
        print("Get Avatars Successful")
        // put avatar into avatars array
		....
		
    case .failure(let error):
        print(error)
    }
    
    // leave the dispatch group after the network request is complete and data is parsed
    userListDispatchGroup.leave()
}

// after both network request complete, code inside this closure will be called
// queue: .main means the main queue, always use main queue to update UI
userListDispatchGroup.notify(queue: .main) {
	print("Both get users and get avatars has completed 👌")
	self.tableView.reloadData()
}
```

<br>

The console log looks like this when the code is executed :  
![Tableview update after both network request is done](https://iosimage.s3.amazonaws.com/2018/12-dispatch-group/oklog.png)

Pretty neat huh? Adding few lines of `dispatchGroup.enter()`, `dispatchGroup.leave()` and `dispatchGroup.notify()` gets the job done easily.

We will explain how dispatch group works in very simplified terms below.

<span id="how"></span>

## How Dispatch Group works
Imagine there's an integer variable **count** inside a DispatchGroup object and its value is set to 0 when you initialize it. The count is used to keep track how many tasks are pending.

When you call **.enter()**, the count increase by 1, as 1 task is added.
When you call **.leave()**, the count reduce by 1, as 1 task is finished.

When the count reaches 0, meaning all task is finished, its **.notify()** method will be called and the closure inside will be executed.

For the Get Users and Get Avatars request we mentioned above, the dispatch group state will look like this :  
![Dispatch Group Chart](https://iosimage.s3.amazonaws.com/2018/12-dispatch-group/dispatchGroupChart.png)

It's simple to visualize and use, kudos to Apple developer team.

## A few stuff to take note of

1. You don't necessary have to use main queue for `dispatchGroup.notify(queue:)`, you can use any of the DispatchQueue like background queue etc, just that to update UI usually we will use the main queue.

2. If the `.notify()` method is placed before all of the `.enter()` method, the notify completion handler will be executed before we call `.enter()`, because at that point (when CPU follow line by line until the `.notify()` part) the **count** of the dispatch group is zero. eg:

```swift
let dispatchGroup = DispatchGroup()

dispatchGroup.notify(queue: .main) {
    print("oops already called notify because at this point of code execution flow, the count of dispatch group is 0")
}

// oh shit already executed notify before entering dispatch group
dispatchGroup.enter()
Alamofire.request("https://httpbin.org/get?avatars").validate().responseJSON { response in
    dispatchGroup.leave()
}

dispatchGroup.enter()
Alamofire.request("https://httpbin.org/get?users").validate().responseJSON { response in
    dispatchGroup.leave()
}

```

<br>

3. Be sure to balance out the number of `.enter()` and `.leave()` (ie. number of enter and leave must be same) in your code, if not, the app might crash.

4. Remember to call `.leave()` outside of the `case .success` , if you only call `.leave()` when the network call is successful, the dispatch group will never be finished if one or more of the network request fail. Refer 3.

5. In this post we mentioned `notify()`, its for asynchronous task, meaning the current thread can continue execute code after the `notify(){}` block and then go back to `notify(){}` completion handler when the dispatch group count reach 0. The alternative of `notify()` is using `wait()`, which will block the current thread from executing the code after `wait()` until all the task of dispatch group is done (ie. count reaches 0).


<div class="post-subscribe">
  <div class="post-subscribe-left">
     <h4 data-drip-attribute="headline">Want to level up your iOS development skills?</h4>
    <span style="font-size:0.9rem;"> 
            Sign up below and I'll send you articles just like this about iOS development to your inbox every one week-ish.
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/295054774/submissions" method="post" data-drip-embedded-form="295054774">
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
                <input type="submit" value="Sign me up!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips to make you a better iOS developer.<br>
                    No Spam. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>