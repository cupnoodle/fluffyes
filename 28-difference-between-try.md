# Difference between Try, Try? and Try!



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


What does the keyword **try** means? Why do you need to use **try** for certain method?



# Try

**try** indicates that a method will throw an error. If you Command + Click the **.decode** method and select 'Jump to Definition', you would see the .decode function has a **throws** keyword. The **throws** means that this function may throw an error and you have to use **try** to handle the potential error.



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



Pay attention to the `throws -> T` in the function and also the comment **throws: `DecodingError.dataCorrupted` if values requested from the payload are corrupted, or if the given data is not valid JSON.**  



// why throw/try is needed? to inform user an error is occurred, instead of returning nil which leave user confused.