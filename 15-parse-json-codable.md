# Parse JSON into struct / object using Decodable protocol



Bless Apple for introducing the **Decodable** , **Encodable**, **Codable** protocol in Swift 4, it makes life as an iOS developer easier phew. 

An object / struct that conforms to **Encodable** protocol can be converted **to** JSON, like this :
`let encodedJSONData = try? JSONEncoder().encode(car)`

An object / struct that conforms to **Decodable** protocol can be converted **from** JSON, like this :
`let car = try? JSONDecoder().decode(Car.self, from: jsonData)`

**Codable** protocol is a combination of **Encodable** + **Decodable**, an object / struct conforming to Codable protocol can be converted from and to JSON.

In this post we will look into how to use the **Decodable** protocol to parse JSON into Object / Struct , parse struct / object into JSON and various scenario of parsing.

Table of Contents:
1. [Parse JSON into single struct / object](#single)
2. [Parse JSON into array of structs / objects
  ](#array)
3. [Parse JSON from non-root key](#non-root)
4. [Parse JSON key into different property name](#different)
5. [Parse JSON with nested struct / object](#nested)
6. [Parse JSON with possible null value](#null)
7. [Cheatsheet for Parsing JSON](#cheatsheet)


<span id="single"></span>

## Parse JSON into single struct / object
Lets start with a simple **Car** struct like this :  
```swift
struct Car: Decodable
{
  let name: String
  let horsepower: Int
}
```

<br>

Notice the struct conforms to the Decodable protocol, so that JSON can be converted into this struct instance.

Assuming we have a JSON like this:
```json
{
  "name": "Toyota Prius",
  "horsepower": 1
}
```
<br>

We can then parse this JSON into Car struct like this:  
```swift
let url = URL(string: "https://demo0989623.mockable.io/car/1")!

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
  
  // Parse JSON into Car struct using JSONDecoder
  guard let car = try? JSONDecoder().decode(Car.self, from: data) else {
    print("Error: Couldn't decode data into car")
    return
  }
  
  // 'gotten car is Car(name: "Toyota Prius", horsepower: 1)'
  print("gotten car is \(car)")
}

// execute the HTTP request
task.resume()
```

<br>

**JSONDecoder** will decode the JSON data into Car struct, **Car.self** tells JSONDecoder that this JSON contains properties of Car struct and decode it into Car struct.

There is a **try?** in front of JSONDecoder as the data you pass to the decoder might be an invalid JSON, or not even a JSON at all. When these happen, `JSONDecoder().decode` method will throw an error. The **Try?** is used to catch error if error is thrown.

Parsing JSON into a single struct object is quite straightforward.

<span id="array"></span>

## Parse JSON into array of structs / objects
Lets use back the **Car** struct:  
```swift
struct Car: Decodable
{
  let name: String
  let horsepower: Int
}
```

<br>

And a JSON containing an array of cars :
```json
[
  {
    "name": "Toyota Prius",
    "horsepower": 1
  },
  {
    "name": "Tesla 3",
    "horsepower" : 3
  },
  {
    "name": "Ferrari",
    "horsepower" : 999
  }
]
```

<br>

We can parse this JSON into an array of car structs like this :  
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

Notice the `JSONDecoder().decode([Car].self, from: data)` , **[Car]** means array of Car, **[Car].self** tells the JSONDecoder that the JSON Data contains type of Array of Car struct. If **Car** conforms to Decodable, **[Car]** is Decodable and can be decoded as well.

<span id="non-root"></span>

## Parse JSON from non-root key
Lets use back the **Car** struct:  
```swift
struct Car: Decodable
{
  let name: String
  let horsepower: Int
}
```

<br>

Similar to previous JSON, we have an array of cars, but now they are inside the key of "**cars**" instead of on the root (most outer) of the JSON :

```json
{
"cars": [
  {
    "name": "Toyota Prius",
    "horsepower": 1
  },
  {
    "name": "Tesla 3",
    "horsepower" : 3
  },
  {
    "name": "Ferrari",
    "horsepower" : 999
  }
]
}
```

<br>

We can parse this JSON into a dictionary which contain an array of car structs like this : 

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
  
  // Parse JSON into Dictionary that contains Array of Car struct using JSONDecoder
  guard let carsArrDict = try? JSONDecoder().decode([String: [Car]].self, from: data) else {
    print("Error: Couldn't decode data into dictionary of array of cars")
    return
  }
  
  // if you are sure the key is "cars"
  let cars = carsArrDict["cars"]!
			
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

This might start to get confusing, we will try explain **[String: [Car]].self**.  

`[Car]` means an array of Car objects.  

`[Int: String]` means a dictionary with Int as Keys and String as Values, eg:  
`var numDict:[Int:String] = [1:"One", 2:"Two", 3:"Three"]`

`[Int: Car]` means a dictionary with Int as Keys and Car as Values, eg:  
```swift
let carOne = Car(name: "Tai Lopez Lamborghini", horsepower: 88)
let carTwo = Car(name: "Toyota Highlander", horsepower: 20)
var carDict:[Int: Car] = [1: carOne, 2: carTwo]
```

<br>

`[String: [Car]]` means a dictionary with String as Keys and Array of Cars as Values, eg:
```swift
let carOne = Car(name: "Tai Lopez Lamborghini", horsepower: 88)
let carTwo = Car(name: "Toyota Highlander", horsepower: 20)
var carArrayDict:[String: [Car]] = ["cars": [carOne, carTwo]]
```

<br>

`[String: [Car]].self` tells the JSONDecoder to decode it into a dictionary of array of cars.

Now we know that `JSONDecoder().decode([String: [Car]].self, from: data)` will parse the JSON data into a dictionary of array of cars like this : `["cars": [carOne, carTwo]]`.

To access the cars array from the dictionary, we use `carsArrDict["cars"]!`. "**cars**" is the key in the JSON. 

There is an exclaimation mark behind as compiler doesn't know if the dictionary has the key "cars", if the dictionary doesn't have the key "cars", it will return nil when we call `carsArrDict["cars"]`. Since we are sure that the JSON contain the key "cars", we can force unwrap it using `!`.

<span id="different"></span>

## Parse JSON key into different property name

This time we will add a new property **manufacturedAt** to the Car struct to indicate which year the car was manufactured : 

```swift
struct Car: Decodable
{
  let name: String
  let horsepower: Int
  let manufacturedAt: Int
}
```

<br>

And the JSON with this format:  
```json
{
  "name": "Fujiwara Tofu Shop Toyota AE86",
  "horsepower": 100,
  "manufactured_at": 1985 
}
```

<br>

Notice that in the JSON, it's **manufactured_at** (with an underscore) but in the Car struct, it's **manufacturedAt** (camelcase).

Using the first example decoding code:  
```swift
let url = URL(string: "https://demo0989623.mockable.io/car/1")!

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
  
  // Parse JSON into Car struct using JSONDecoder
  guard let car = try? JSONDecoder().decode(Car.self, from: data) else {
    print("Error: Couldn't decode data into car")
    return
  }
  
  // 'gotten car is Car(name: "Toyota Prius", horsepower: 1)'
  print("gotten car is \(car)")
}

// execute the HTTP request
task.resume()
```

<br>

If you try to run the code with the new JSON / Car Struct, you will get a decoding error as there is no **manufactured_at** property in the Car struct.

![Error decoding Car](https://iosimage.s3.amazonaws.com/2018/15-parse-json-codable/error_decode.png)

To solve this, we will add some code to tell the compiler to map **manufactured_at** from the JSON to **manufacturedAt** of the Car struct :  

```swift
struct Car: Decodable
{
  let name: String
  let horsepower: Int
  let manufacturedAt: Int
	
  // map 'manufactured_at' from JSON to 'manufacturedAt' of Car
  // 'name' and 'horsepower' are left alone as JSON already have them
  enum CodingKeys : String, CodingKey {
    case name
    case horsepower
    case manufacturedAt = "manufactured_at"
  }
}
```

<br>

By default, if you didn't declare the **CodingKeys** enum (we didn't declare this for the past few examples),  compiler will auto generate the enum CodingKeys for you, which will map all the Keys of JSON directly to the Car struct without change (eg: '**name**' from JSON to '**name**' of Car struct, '**horsepower**' from JSON to '**horsepower**' of Car struct).

Since we are not using the default direct mapping, as we have a different property name than the JSON key ('**manufactured_at**' from JSON to '**manufacturedAt**' of Car struct). We will have to override the default CodingKeys by declaring it explicitly:

```swift
// map 'manufactured_at' from JSON to 'manufacturedAt' of Car
// 'name' and 'horsepower' are left alone as JSON already have them
enum CodingKeys : String, CodingKey {
  case name
  case horsepower
  case manufacturedAt = "manufactured_at"
}
```

<br>

**CodingKeys** is an [enum](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html#//apple_ref/doc/uid/TP40014097-CH12-ID146) with type of String and conforms to the protocol [CodingKey](https://developer.apple.com/documentation/swift/codingkey). Its type is string as JSON keys are usually string (eg: `{"name": "GTR", "horsepower" : 250}`).

**CodingKey** protocol defines how properties of a struct / object are linked to its encoded form (eg: JSON).

Note: 
```swift
// if we didn't specify the case value, then it
enum CodingKeys : String, CodingKey {
  case name
  case horsepower
  case manufacturedAt = "manufactured_at"
}

// is equivalent to
enum CodingKeys : String, CodingKey {
  case name = "name"
  case horsepower = "horsepower"
  case manufacturedAt = "manufactured_at"
}

```

<br>

Note 2: If we decide to override the **CodingKeys** enum, we have to specify **all** of the properties of a struct / class in **CodingKeys** enum. Even if we missed just one, compiler will complain :  

![Coding Keys error as missing one property 'name'](https://iosimage.s3.amazonaws.com/2018/15-parse-json-codable/coding_key_error.png)

The error message `Type 'Car' does not conform to protocol 'Decodable'` doesn't inform you that the missing property in the CodingKeys needs to be filled. This might cause confusion for developer who are new to Decodable / Codable protocol 😞.

Feeling difficult to remember how to override JSON property key?

<a href="https://www.getdrip.com/forms/200274343/submissions/new"  target="_blank" data-drip-show-form="448009646" style="background-color:#4baa33; color:#fff; padding: 1rem; border-radius:4px;">Send me the JSON Parsing Cheatsheet</a>

<span id="nested"></span>

## Parse JSON with nested struct / object

Lets say a **manufacturer** key is added into the car JSON and it contains the car manufacturer information such as name and country of origin, like this: 

```json
{
  "name": "Prius",
  "horsepower": 100,
  "manufacturer" : {
    "name": "Toyota",
    "country": "Japan"
  }
}
```

<br>

We can create another struct named "Manufacturer" and add it to the Car struct like this :  

```swift
// Manufacturer.swift
struct Manufacturer: Decodable
{
  let name: String
  let country: String
}
```
<br>

```swift
// Car.swift
struct Car: Decodable
{
  let name: String
  let horsepower: Int
  let manufacturer: Manufacturer
}
```
<br>

Then we just parse it using JSONDecoder as usual:  
```swift
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
  
  // Parse JSON into Car struct
  guard let car = try? JSONDecoder().decode(Car.self, from: data) else {
    print("Error: Couldn't decode data into car")
    return
  }
  
  // car is Car(name: "Prius", horsepower: 100, manufacturer: codable.Manufacturer(name: "Toyota", country: "Japan"))
  print("car is \(car)")
  
  // car manufacturer: Manufacturer(name: "Toyota", country: "Japan")
  print("car manufacturer: \(car.manufacturer)")
}

// execute the HTTP request
task.resume()
```

<br>

Parsing a nested struct / object is straightforward and no additional code is required on the parsing part, just add the child struct / object property to the parent.

Note:
I have removed the **manufacturedAt** property from Car struct to isolate away the CodingKeys from the previous section, in order to simplify the explanation of this section.

If you are wondering how to set the CodingKeys for a struct which have a child struct, you just need to specify the property of the parent struct (no need to worry about property of the child struct), like this :  
```swift
// Car.swift
struct Car: Decodable
{
  let name: String
  let horsepower: Int
  let manufacturedAt: Int
  let manufacturer: Manufacturer
	
  enum CodingKeys: String, CodingKey{
    case name
    case horsepower
    case manufacturedAt = "manufactured_at"
    case manufacturer
  }
}
```

<br>

```swift
// Manufacturer.swift
struct Manufacturer: Decodable
{
  let name: String
  let country: String
}

// No need to set / override CodingKeys as keys (eg: 'name', 'country') of manufacturer in the JSON match the name of properties
```

<br>

<span id="null"></span>

## Parse JSON with possible null value

Sometimes you might need to deal with JSON that have null values, lets add a key **fuel_tank_capacity** into the JSON to indicate how many liter of fuel the car fuel tank can hold:  
```json
[
  {
    "name": "Volkswagen Bettle",
    "horsepower": 1,
    "fuel_tank_capacity": 40
  },
  {
    "name": "Tesla Model S",
    "horsepower" : 3,
    "fuel_tank_capacity": null
  }
]
```

<br>

Since [Tesla](https://en.wikipedia.org/wiki/Tesla_Model_S) is an electric car without fuel tank, the API returns null for the fuel_tank_capacity.

Then we add **fuelTankCapacity** property to the Car struct like this:  
```swift
struct Car: Decodable
{
  let name: String
  let horsepower: Int
  let fuelTankCapacity: Int
  
  enum CodingKeys: String, CodingKey{
    case name
    case horsepower
    case fuelTankCapacity = "fuel_tank_capacity"
  }
}
```

<br>

Lets use back the JSONDecoder code from the parsing array section :  
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
  
  // Parse JSON into Array of Car struct using JSONDecoder
  guard let cars = try? JSONDecoder().decode([Car].self, from: data) else {
    print("Error: Couldn't decode data into cars array")
    return
  }
  
  for car in cars {
    print("car name is \(car.name)")
    print("car horsepower is \(car.horsepower)")
    print("car fuel tank capacity is \(car.fuelTankCapacity)")
    print("---")
  }
}

// execute the HTTP request
task.resume()
```

<br>

If you run the code above, compiler will complain that it can't decode the JSON  :

![Can't decode null value](https://iosimage.s3.amazonaws.com/2018/15-parse-json-codable/error_decode_2.png)

Compiler is unable to decode because in the Car struct, `let fuelTankCapacity: Int` tells the compiler to expect an integer value, but the JSON returned a `null` instead. 

`null` in JSON is equivalent to `nil` in Swift. To handle `null` from JSON, we have to make the **fuelTankCapacity** property optional like this :  

```swift
struct Car: Decodable
{
  let name: String
  let horsepower: Int
  
  // fuel tank capacity might be null
  let fuelTankCapacity: Int?
  
  enum CodingKeys: String, CodingKey{
    case name
    case horsepower
    case fuelTankCapacity = "fuel_tank_capacity"
  }
}
```

<br>

If `null` is found from the JSON, it will be converted to `nil` in Swift.
We then modify the print statements of cars to become like this :  
```swift
for car in cars {
  print("car name is \(car.name)")
  print("car horsepower is \(car.horsepower)")
  
  // only print fuelTankCapacity if it exist (ie. not null)
  if let fuelTankCapacity = car.fuelTankCapacity {
    print("car fuel tank capacity is \(fuelTankCapacity)")
  }
  
  print("---")
}
```

<br>

Then we run the code again and got this nice output:  
![Print cars log](https://iosimage.s3.amazonaws.com/2018/15-parse-json-codable/print_cars.png)

<br>

Bonus note:  
By setting a property as optional, the compiler can also handle cases where the key of property doesn't exist in the JSON, like this :  
```json
[
  {
    "name": "Volkswagen Bettle",
    "horsepower": 1,
    "fuel_tank_capacity": 40
  },
  {
    "name": "Tesla Model S",
    "horsepower" : 3
  }
]
```

<br>

Notice that **fuel_tank_capacity** key is absent for the Tesla Model car, if we run the same parsing code, we will still get a nice output as compiler will set the property value as `nil` if the property key doesn't exist in the JSON.

![Print cars log](https://iosimage.s3.amazonaws.com/2018/15-parse-json-codable/print_cars.png)

<span id="cheatsheet"></span>

## Cheatsheet
Phew this is a long post 😅.
To save you time from Googling "Parse json array swift" etc, I have made a handy cheatsheet that you can quickly refer for different cases of JSON parsing I have mentioned above.
<div class="post-subscribe" style="margin-top:1rem;">
  <div class="post-subscribe-left">
    <h4> Get the Cheatsheet for parsing JSON in Swift</h4>
    <span> 
            <img src="https://iosimage.s3.amazonaws.com/2018/15-parse-json-codable/cheatsheet_preview.png"></img>Save yourself from googling "parse json array swift" every time you deal with JSON
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/200274343/submissions" method="post" data-drip-embedded-form="200274343">
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
                <input type="submit" value="Send me the cheatsheet!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">Bi-weekly ish iOS Development tips. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>
​    