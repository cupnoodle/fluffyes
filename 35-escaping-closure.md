# What is escaping closure and when to use it

> What does "Closure use of non-escaping parameter may allow it to escape" means?!



In the post [How to return value from closure](https://fluffy.es/return-value-from-a-closure/), we passed a closure (completionHandler) into the function of fetching user from Web API , so that we can call this closure when the user data has been fetched.



```swift
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

override func viewDidLoad() {
  // fetch the user data from web api, 
  // once the data is retrieved, execute the code inside userCompletionHandler
  fetchUser(userID: 1, userCompletionHandler: { user, error in
    if let user = user {
      print("user first name is \(user.first_name)")
    }
  })
}
```

<br>



Notice that there's an **@escaping** keyword after the userCompletionHandler, what does it do? If we remove it, Xcode will complain to us: 



![closure error](https://iosimage.s3.amazonaws.com/2018/35-escaping-closure/closureError.png)



"Closure use of non-escaping parameter may allow it to escape"



What does this error means? To understand this, we will first need to know about non-escaping, which is the default closure behaviour.



## Default closure behaviour (non-escaping)

Say for a simple function with closure like this :

```swift
func macICanBuy(budget: Int, closure: (String) -> Void) {
  print("checking budget...")
  
  closure("Mcdonalds' Big Mac")
  
  print("macICanBuy finished execution")
}

override func viewDidLoad(){
  macICanBuy(budget: 100, closure: { mac in
    print("I can afford a \(mac)")
  })
}

// output: 
// checking budget...
// I can afford a Mcdonalds' Big Mac
// macICanBuy finished execution
```

<br>

Before the app executes the **macICanBuy** function, it will load the passed parameter (budget and closure) into the phone memory (RAM) so the function can use these data.



After the app finish execute the **macICanBuy** function, the app no longer needs the budget and closure data, hence it will remove them from the memory.  The closure data didn't manage to escape from being removed from memory after the function has finished executing, hence it is called '**non-escaping**' closure. 



![nonescaping](https://iosimage.s3.amazonaws.com/2018/35-escaping-closure/nonescapingMemory.png)



## Escaping closure behaviour

Let's modify the previous function a bit. Inside the **macICanBuy** function. We wrap the closure inside **DispatchQueue.main.asyncAfter()** so that the closure will be executed two seconds after the current time. We will also need to add **@escaping** here else Xcode will complain.



```swift
func macICanBuy(budget: Int, closure: @escaping (String) -> Void) {
  print("checking budget...")
  
  // execute the closure 2 seconds after current time
  DispatchQueue.main.asyncAfter(deadline: .now() + 2, execute: {
    closure("Mcdonalds' Big Mac")
  })
  
  print("macICanBuy finished execution")
}

override func viewDidLoad(){
  macICanBuy(budget: 100, closure: { mac in
    print("I can afford a \(mac)")
  })
}

// output:
// checking budget...
// macICanBuy finished execution
// I can afford a Mcdonalds' Big Mac
```

<br>



Notice the order of the output, since we tell the app to only execute the closure 2 seconds after current time, the app will continue executing the next line first then back to the closure.



![order](https://iosimage.s3.amazonaws.com/2018/35-escaping-closure/order.png)



After the line **print("macICanBuy finished execution")** is executed, the function has reached the last line / end of function, then the app will mark the function as finished execution. But remember that the closure will only execute 2 seconds after? How does the app remember the closure data after the function has finished execution?



Before the app executes the **macICanBuy** function, it will load the passed parameter (budget and closure) into the phone memory (RAM) so the function can use these data.



After the app finish execute the **macICanBuy** function, the app no longer needs the budget data and it will remove budget from the memory, but the app didn't remove the closure data as it still needs the closure data to execute it 2 seconds after the **macICanBuy** function has finished execution.  The closure data has managed to escape from being removed from memory even when the function has finished execuction, hence it is called '**escaping**' closure. 



![escapingMemory](https://iosimage.s3.amazonaws.com/2018/35-escaping-closure/escapingMemory.png)



The closure data will only be removed from memory after the closure has finished executed.



Xcode shows us the error message "Closure use of non-escaping parameter may allow it to escape" when we didn't put **@escaping** for closure that will be executed AFTER the function has finished executing.



## Why Swift forces us to put the keyword @escaping

As this point, you might wonder

> Wait a minute, since Xcode already knew the closure will be escaping (it shows us error when we didnt put @escaping), why can't Xcode just understood and deal with it instead of forcing me to put @escaping? Why the fuss?



Andrew has written a [great explanation](https://www.andrewcbancroft.com/2017/05/11/why-do-we-need-to-annotate-escaping-closures-in-swift/) on this behaviour. In short, Swift forces you to put **@escaping** to remind yourself that the closure will be executed asynchronously / in the future (not now). This serve as a reminder so you won't get confused like "Why this certain output only appear 3 seconds after I execute the function?".



## When to use @escaping



