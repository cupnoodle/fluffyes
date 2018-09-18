# Difference between Try, Try? and Try!

> What does try? means? When to use these

This post assume you have some knowledge on [Optionals](https://fluffy.es/eli-5-optional/).



You might have came across the keyword `try` when trying to [parse JSON retrieved from API](https://fluffy.es/parse-json-using-decodable-protocol/) or other instances. 



For example : 

```swift
let url = URL(string: "https://demo0989623.mockable.io/cars")!

let task = URLSession.shared.dataTask(with: url) { data, response, error in
    
    // ensure there is no error for this HTTP response
    guard error == nil else {
        print ("error: \(error!)")
        return
    }
    
    // ensure there is data returned from this HTTP response
    guard let data = data else {
        print("No data")
        return
    }
    
    // Parse JSON into array of Car struct using JSONDecoder
    // Notice the try? here
    guard let cars = try? JSONDecoder().decode([Car].self, from: data) else {
        print("Error: Couldn't decode data into cars array")
        return
    }
    
    for car in cars {
        print("car name is \(car.name)")
        print("car horsepower is \(car.horsepower)")
        print("---")
    }
}

// execute the HTTP request
task.resume()

```

<br>
What does the keyword **try** means? Why do you need to use **try** for certain method?



# Try

**try** indicates that a method might throw an error. If you Command + Click the **.decode** method and select 'Jump to Definition', you would see the .decode function has a **throws** keyword. The **throws** means that this function may throw an error and you have to use **try** to handle the potential error.



![decode](https://iosimage.s3.amazonaws.com/2018/27-try/commandClick.png)



You can see the JSONDecoder().decode function declaration and its comment : 

```swift
/// Decodes a top-level value of the given type from the given JSON representation.
///
/// - parameter type: The type of the value to decode.
/// - parameter data: The data to decode from.
/// - returns: A value of the requested type.
/// - throws: `DecodingError.dataCorrupted` if values requested from the payload are corrupted, or if the given data is not valid JSON.
/// - throws: An error if any value throws an error during decoding.

open func decode<T>(_ type: T.Type, from data: Data) throws -> T where T : Decodable
```

<br>



`throws` indicate that the decode function might throw an error. `-> T` means it will return a type **T** which you have passed in, eg: it will return the type of **[Car]** which we have passed into **.decode([Car].self)** .



The function implementation might look like this (not the actual code, just my guess as the implementation is not open source):

```swift
func decode<T>(_ type: T.Type, from data: Data) throws -> T {
  // if data is not a valid JSON
  if(!data.isValidJSON){
    // throw error and stop execution of this function
    throw DecodingError
  } 
  
  // parse JSON data here
  let parsedData = parseData(data)
  return parsedData
}
```

<br>

As this function might throw an error (eg: DecodingError), we will need to use a **try** and a **do / catch** statement to handle the error like this : 

```swift
// this URL contains an invalid JSON
let url = URL(string: "https://demo0989623.mockable.io/car/invalid")!

let task = URLSession.shared.dataTask(with: url) { data, response, dataTaskError in
    
    // ensure there is no error for this HTTP response
    guard dataTaskError == nil else {
        print ("error: \(dataTaskError!)")
        return
    }
    
    // ensure there is data returned from this HTTP response
    guard let data = data else {
        print("No data")
        return
    }
    
    // Parse JSON into array of Car struct using JSONDecoder
    do {
      let cars = try JSONDecoder().decode([Car].self, from: data)
    } catch {
      // If there is any error thrown, the 'catch' block will catch the error
      // Swift will create and use a local constant named 'error' to store the error thrown by the function
      print("error \(error)")
    }
}

// execute the HTTP request
task.resume()
```



Since the URL does not contain a valid JSON, an error will be thrown and it will be handled by the **catch** block. The error will be stored in a local constant named **error** inside the catch block. ([https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html#ID541](https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html#ID541))



If you run the code above, you will see the following error in your console log : 

![JSON Error](https://iosimage.s3.amazonaws.com/2018/27-try/jsonError.png)



This is the **DecodingError** thrown by the **JSONDecoder.decode()** function we mentioned earlier.



This is the basic of error handling using **try** and **do / catch** block, you can read more about error handling in the [Official Swift documentation here](https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html).



**try** is usually used when you want to handle the errors manually.



## Try?

Similar to `try`, you can use **try?**  on function that might throw error too. The difference is that you won't need to put `try?` inside a do/catch block. You can use it directly like this :

```swift
let cars = try? JSONDecoder().decode([Car].self, from: data)
```

<br>

If the **JSONDecoder.decode()** function return a value successfully without error, that value will be saved into **cars** as usual. 



If **JSONDecoder.decode()** throws an error, the error will be ignored and **cars** will be assigned a nil value.


For the example below, as the URL contains an invalid JSON, you will see the console log outputs '*cars is nil because JSONDecoder().decode thrown an error*' .

```swift
// this URL contains an invalid JSON
let url = URL(string: "https://demo0989623.mockable.io/car/invalid")!

let task = URLSession.shared.dataTask(with: url) { data, response, dataTaskError in
    
    // ensure there is no error for this HTTP response
    guard dataTaskError == nil else {
        print ("error: \(dataTaskError!)")
        return
    }
    
    // ensure there is data returned from this HTTP response
    guard let data = data else {
        print("No data")
        return
    }
    
    // Parse JSON into array of Car struct using JSONDecoder
    if let cars = try? JSONDecoder().decode([Car].self, from: data) {
      // do stuff with the parsed cars data here
      
    } else {
      print("cars is nil because JSONDecoder().decode thrown an error")
    }
}

// execute the HTTP request
task.resume()
```

<br>



If you don't care much about the error handling and just want to focus on the value returned, **try?** is a good way to do it.



## Try!

When there's an exclaimation mark in the syntax, it means something horrible can happen. Similar to `try?`, **try!** doesn't need to be put inside do/catch block as well. The difference is that when an error is thrown, **your app will crash** instead of returning nil. This is similar to unwrapping a nil value in optionals.



```swift
// Only do this if you are very very sure that the .decode function below won't throw an error
let cars = try! JSONDecoder().decode([Car].self, from: data)

// this is equivalent to
let optionalCars = try? JSONDecoder().decode([Car].self, from: data)
let cars = optionalCars!
```

<br>



If you are **absolutely sure** that the function won't throw an error, you can use **try!** . Usually I will avoid using anything that has an exclaimation mark **!** in code, better be safe than sorry.