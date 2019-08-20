# Understanding Reference and Value type



Recently I have stumbled across a Reddit thread in r/swift, where a user is confused about why the output of his code does not match his expectation : 

```swift
class Member {
    var memberID: Int? = nil
}

// Array of "Member" class
var members = [Member]()

// temporary variable for member
var memberHolder = Member()

for i in 1...5 {
    memberHolder.memberID = i
    members.append(memberHolder)
}

for m in members {
    print("member ID is \(m.memberID)")
}
```



He was expecting the output to be like this :

```
member ID is Optional(1)
member ID is Optional(2)
member ID is Optional(3)
member ID is Optional(4)
member ID is Optional(5)
```



But the actual output from Xcode is that all member has the same memberID! 😱🤔

```
member ID is Optional(5)
member ID is Optional(5)
member ID is Optional(5)
member ID is Optional(5)
member ID is Optional(5)
```



If you also got surprised by the actual output, you might not have a good grasp of reference type in Swift, which this article will cover below, read on!



## Memory Address

Before going into explaining the difference between reference and value type, I think it is crucial to know the concept of Memory Address.



Your computer and iOS devices has a component named RAM (Random Access Memory), which is used to store working data when you are using the device. For example, when you are playing a game, there will be a loading screen to load the graphic assets and audio needed in the level into the memory (RAM), then when you are playing that level, the game will access the graphic / audio / character information from the RAM.



For your Mac, the RAM refer to the 'Memory' section.

![memory size](https://iosimage.s3.amazonaws.com/2019/61-reference-value-type/memorySize.png)



For most modern computers, RAM / Memory segments its space (eg: 8GB space) into discrete chunks (multiple 1-byte space), and each chunk has an unique memory address. Below is a diagram showing different chunk (with different memory address) holding different text data : 

![ram address](https://iosimage.s3.amazonaws.com/2019/61-reference-value-type/ramaddress.png)



Memory address usually is a hexadecimal string (eg: 0x12345) in most computer system, when the CPU need to get a data from a certain chunk of memory, it will reference the memory address. This concept is also known as [Pointer](https://en.wikipedia.org/wiki/Pointer_(computer_programming)) in Computer Science.



When we declare variables in Swift / Objective-C , the operating system will allocate a (few) chunk of memory (with memory address) to store them.



## Value types

Some examples of value type include primitive types such as Int, Double, String ( in Swift) , tuples, enum and struct. Value type instances (ie. `var myMoney: Int = 500`) create a new copy of the value assigned instead of storing a reference to the value.



```swift
var yourMoney : Int = 500
var myMoney : Int = yourMoney

// you lost 50 dollar
yourMoney = yourMoney - 50

print("yourMoney is \(yourMoney)") // 450
print("myMoney is \(myMoney)") // 500
```



![value type demo](https://iosimage.s3.amazonaws.com/2019/61-reference-value-type/valueTypeDemo.png)



Despite the myMoney value is assigned from yourMoney value, **myMoney** 's value remain unchanged even though yourMoney has decreased. This is because during the assignment `myMoney = yourMoney` , the value '500' is copied over.



## Reference types

Class, Closure and Functions are reference type. When a reference type variable is declared (eg: `var dog : Dog = Dog()`), the system will allocate a chunk of memory space in RAM, and store the memory address of the Dog object to the variable. **The variable is actually storing the memory address** of the Dog data, not the Dog data itself. When another class object is assigned to an existing class object, the memory address is copied over, not the data.



```swift
class Dog {
    var name : String
  
    init(name: String) {
      self.name = name
    }
}

var labrador = Dog(name: "labrador")
print("dog name is \(labrador.name)") // "dog name is labrador"

var schnauzer = labrador
print("schnauzer name is \(schnauzer.name)") // "schnauzer name is labrador"

schnauzer.name = "schnauzer"

print("dog name is \(labrador.name)") // dog name is schnauzer
```



![reference type diagram](https://iosimage.s3.amazonaws.com/2019/61-reference-value-type/reference.png)



The diagram above shows how reference types work, as both "labrador" and "schnauzer" variables point to the same memory address, modifying the **.name** on either one of them will affect the same name data stored on the memory address.



In Swift, we can [print out the memory address of reference-type variables](https://stackoverflow.com/a/41666807/1901264) using **Unmanaged.passUnretained(variable).toOpaque()**  like this : 

```swift
print("memory address of labrador is \(Unmanaged.passUnretained(labrador).toOpaque())")
print("memory address of schnauzer is \(Unmanaged.passUnretained(schnauzer).toOpaque())")
```



// in objective-C, print memory address



Here's the output if we execute the code above in Playground :

![memory address output](https://iosimage.s3.amazonaws.com/2019/61-reference-value-type/memoryAddressProof.png)



Notice that the memory address of labrador and schnauzer is exactly same! (0x00007fa409f3cb30)



Next time if you find yourself confused about how reference type work or why a class object variable output value isn't as you expected, just remember that reference type copies memory address and modify the same chunk of data.



None of the article about Swift value/reference type I found online has explained about the memory address aspect





// show the solution to the question at top











