# How to return value from a closure?

> How do I return data from URLSession dataTask ?

> Why do I get **nil** when I return the data from URLSession dataTask? I confirm that the json is retrieved!

> Why do I get error **unexpectedly found nil while unwrapping an Optional value** when I return the data?

> Wtf is **It is expecting non-void return statement in void function** ?!


Previously I have written a [tutorial for using URLSession](https://fluffy.es/nsurlsession-urlsession-tutorial/) , then I surveyed around in Reddit / StackOverflow and found quite some people stuck on how to return the data / value retrived from the web API using URLSession.

This post will focus on the context of using URLSession dataTask, but it can also be applied to other function that has completionHandler. [Click here to jump straight to the answer](#answer) if you want to skip explanation of questions listed above.

Lets say you have a function to get user data like this :
```swift
override func viewDidLoad() {
  super.viewDidLoad()
  // Get user with ID 2
  let user = fetchUser(userID: 2)
  print("user first name is : \(user?.firstName)")
  nameLabel.text = user?.firstName
}

func fetchUser(userID: Int) -> User? {
  let url = URL(string: "https://reqres.in/api/users/\(userID)")!
  let task = URLSession.shared.dataTask(with: url, completionHandler: { data, response, error in
      
    guard let data = data else { return }
    do {
      // parse json data and return it
      let decoder = JSONDecoder()
      let jsonDict = try decoder.decode([String: User].self, from: data)
      if let userData = jsonDict["data"] {
        return userData
      }
        
    } catch let parseErr {
      print("JSON Parsing Error", parseErr)
    }
  })
  
  task.resume()
}
```

<br>


The code above won't compile as you will get error mesage like this : 
![Unexpected non-void return value in void function](https://iosimage.s3.amazonaws.com/2018/14-return-value-closure/unexpected-void.png)

"Wait.. I already mentioned to return **User** type in the `fetchUser` function, why does it tell me that **Unexpected non-void return**? I should return void?"

The problem is that the `return userData` is inside the `completionHandler` closure. Remember **completionHandler** parameter accept a closure (function as variable)? If the return is inside the completionHandler, it will return value for the function of completionHandler, not the outer `fetchUser` function.

At below, we can see that completionHandler expects a function that accepts _Data?, URLResponse?, Error?_ type as parameters and return void.
![Completion handler expected void](https://iosimage.s3.amazonaws.com/2018/14-return-value-closure/completionHandler.png)

If you return a value other than void in the function you passed to completionHandler, it doesn't match the type of completionHandler, hence compiler complains.

"Ahh! I should move the `return userData` to outside of the completionHandler?"

Lets move the `return` statement to outside of the completionHandler like this :  

```swift
func fetchUser(userID: Int) -> User? {
  
  var user : User?
  let url = URL(string: "https://reqres.in/api/users/\(userID)")!
  let task = URLSession.shared.dataTask(with: url, completionHandler: { data, response, error in

    guard let data = data else { return }
    do {
      // parse json data and return it
      let decoder = JSONDecoder()
      let jsonDict = try decoder.decode([String: User].self, from: data)
      if let userData = jsonDict["data"] {
        // save the userData to the outer variable user
        user = userData
      }
      
    } catch let parseErr {
      print("JSON Parsing Error", parseErr)
    }
  })
  
  task.resume()
  // return user outside the completion handler
  return user
}

```

<br>

The code compiles but the **user firstName** is nil and further investigation shows that the **user** returned is nil, wait what?
![User first name is nil](https://iosimage.s3.amazonaws.com/2018/14-return-value-closure/userFirstNameNil.png)

This is because `task.resume()` is **asynchronous** , the line below `task.resume()` will be immediately executed without having to wait for the HTTP response to arrive (ie. before executing the code inside `completionHandler`). The code execution flow will look like this : 
![Asynchronous flow](https://iosimage.s3.amazonaws.com/2018/14-return-value-closure/asynchronous.png)

Notice that `return user` is executed before `user = userData` is executed, hence you will get nil from this function.

If you have used something like `user!` somewhere in the code , you will get the error **unexpectedly found nil while unwrapping an Optional value** because user is nil and you force unwrap it with `!`.

"Huh.. since I can't return the user data inside nor outside of the `completionHandler`, how should I access and use the user data after  `URLSession.shared.dataTask` has retrieved data?"

One of the solutions is to add a closure parameter to the outer `fetchUser()` function, which we will explain further below.

<span id="answer"></span>

## Use another closure to use the data received in the completionHandler

Since `task.resume()` is asynchronous, we can't write code in a sequential way to return the user data. We will add a closure parameter `userCompletionHandler` to the `fetchUser` function and also remove the return User ` -> User?` to make it a void function : 

```swift
// add userCompletionHandler and remove ' -> User?' to make it a void function
func fetchUser(userID: Int, userCompletionHandler: @escaping (User?, Error?) -> Void) {
  let url = URL(string: "https://reqres.in/api/users/\(userID)")!
  let task = URLSession.shared.dataTask(with: url, completionHandler: { data, response, error in

    guard let data = data else { return }
    do {
      // parse json data and return it
      let decoder = JSONDecoder()
      let jsonDict = try decoder.decode([String: User].self, from: data)
      if let userData = jsonDict["data"] {
        userCompletionHandler(userData, nil)
      }
      
    } catch let parseErr {
      print("JSON Parsing Error", parseErr)
      userCompletionHandler(nil, parseErr)
    }
  })
  
  task.resume()
  // function will end here and return
  // then after receiving HTTP response, the completionHandler will be called
}
```

<br>

Notice that we have added `@escaping` for the `userCompletionHandler` parameter, this is because the `fetchUser` function will finish execute and return before the `userCompletionHandler()` function is being called. (ie. execute the last line `task.resume()` and reaches the end of the function before the `userCompletionHandler()` is being executed). Still confused? Try looking at the code execution flow image above again, imagine there is an invisible `return` after the last line `task.resume()`. Read more about [escaping closure on Apple documentation here](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html#//apple_ref/doc/uid/TP40014097-CH11-ID546).

We will also update `viewDidLoad()` to use the new `fetchUser()` method :
```swift
override func viewDidLoad() {
  super.viewDidLoad()
  // Do any additional setup after loading the view, typically from a nib.
  
  fetchUser(userID: 2, userCompletionHandler: { user, error in
    
    if let user = user {
      print("user first name is : \(user.firstName!)")
      self.nameLabel.text = user.firstName
    }
  })
}
```

<br>

Then the flow of code execution will be like this :  
![userCompletionHandler Flow](https://iosimage.s3.amazonaws.com/2018/14-return-value-closure/completionHandlerFlow.png)

(I thought of explaning it using words but figured out using picture with arrow diagram will be much faster and clearer, phew)

`userCompletionHandler` function will be called inside the `completionHandler` of `URLSession.shared.dataTask()`. You can pass the user data or relevant error to `userCompletionHandler()`. Then you can access these data in `viewDidLoad()` by calling `fetchUser()`!

Whenever you want to get and use user data from Web API, just call `fetchUser()` and use its `userCompletionHandler : { user, error in }`.

Hope the picture above also gave you an idea on how completion handler works 😆.

<div class="post-subscribe">
  <div class="post-subscribe-left">
     <h4 data-drip-attribute="headline">Want to level up your iOS development skills?</h4>
    <span style="font-size:0.9rem;"> 
            Sign up below and I'll send you articles just like this about iOS development to your inbox (and also subscriber exclusive tips) every one week-ish.
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
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips. <br>Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>