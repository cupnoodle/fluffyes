# ELI 5 - Optional and jargons

> Why would you want to use an optional value?!

> I always click the 'Fix' button provided by Xcode whenever it tells me there's an error about optional, I have no idea what it does actually

![Optional fun fixes](https://iosimage.s3.amazonaws.com/2018/20-optionals/optionalFun.png)

> FFFUUUUUUU fatal error: unexpectedly found nil while unwrapping an Optional value

Ah optional, I've struggled with Optionals when Swift first came out. I started iOS Development using Objective-C and there was no concept of optional in it (just check for `object != nil`). In this post I will try explain why optional is being introduced, what is the use of it, how does it work and various jargon around it.

Table of Contents:  
1. [Why Optional exist](#why)
2. [How does Optional work](#how)
3. [Force unwrap](#force-unwrap)
4. [Optional binding](#optional-binding)
5. [Optional chaining](#optional-chaining)
6. [Nil coalescing](#nil-coalescing)
7. [Implicitly unwrapped optional](#implicit)
8. [Summary](#summary)
9. [Use guard to simplify optionals handling](#guard)


<span id="why"></span>
## Why Optional exist
> Why does Optional exist in Swift? Can't we just use type Object instead of Object? (optional) for all types?

The whole purpose of optional being introduced is to cater for **nil**.
`nil` means "no value", meaning there is nothing at all. If a variable is nil, it has no data inside. A variable with optional type can use `nil` to indicate it has no value.

![nilBox](https://iosimage.s3.amazonaws.com/2018/20-optionals/nilBox.png)

> Why is there a need to put 'nil' into a variable?

Because there will be cases where some variables might not have a value, for example, let's say we have a `Person` struct like this :

```swift
struct Person {
    let firstName: String
    let middleName: String
    let lastName: String
}
```

<br>

As I don't have a middle name, I have to tell the code my middle name does not exist by passing in a nil:  
![Axel](https://iosimage.s3.amazonaws.com/2018/20-optionals/axelNil.png)

Xcode shows error "Nil is not compatible with expected argument type 'String'" as the `Person` struct only accept String value for `middleName`, it doesn't allow nil value.

Of course I could pass an empty string `""` to the middle name, but it would mean that my middle name is an empty string instead of meaning that I don't have a middle name.

To allow nil value for middleName, we have to change the middleName type to optional string `String?` :  

```swift
struct Person {
    let firstName: String
    // make it optional, in case a person doesn't have a middle name
    let middleName: String?
    let lastName: String
}
```

<br>

After the change, I can put `nil` as middle name for the Person struct and it will look like this in Playground:  
![Axel yay](https://iosimage.s3.amazonaws.com/2018/20-optionals/axelNil2.png)

<br>

<span id="how"></span>
## How does Optional work
Continuing from previous example, our current `Person` struct looks like this:  

```swift
struct Person {
    let firstName: String
    // make it optional, in case a person doesn't have a middle name
    let middleName: String?
    let lastName: String
}

let axel = Person(firstName: "Axel", middleName: nil, lastName: "Kee")
```

<br>

If we print out all the names of `axel`, it will look like this:  
![Print nil](https://iosimage.s3.amazonaws.com/2018/20-optionals/printOptional.png)

It print out `nil` for middle name as I don't have a middle name, it works as expected right? Let's say now I have a middle name called 'Cupnoodle' (it's my Github username), if we print it out, we will see something like this:  

![Print optional](https://iosimage.s3.amazonaws.com/2018/20-optionals/optionalWat.png)

Wait... why the middleName `Cupnoodle` is wrapped inside `Optional(...)` ?! 🤔

This is because `Optional` is an enum type, we can verify it by typing 'Optional' in Xcode , then <kbd>Command</kbd> + Click it and select **Jump to Definition**.

![Optional description](https://iosimage.s3.amazonaws.com/2018/20-optionals/optionalDescription.png)

![Jump Definition](https://iosimage.s3.amazonaws.com/2018/20-optionals/jumpDefinition.png)

This is the code implementation of `Optional` :  
![Optional Enum](https://iosimage.s3.amazonaws.com/2018/20-optionals/optionalEnum.png)

Since Swift is open source, you can check the full source code of Optional [on Github here](https://github.com/apple/swift/blob/master/stdlib/public/core/Optional.swift).

When you are defining an optional variable, you are actually defining an Optional enum:  

```swift
// eg. optional with value
let firstName : String? = "Axel"
// is actually calling
let firstName : Optional<String> = Optional.some("Axel")


// eg. optional with nil
let middleName : String? = nil
// is actually calling
let middleName : Optional<String> = Optional.none
```

<br>

When you are printing an optional value, you will get output like `Optional("Cupnoodle")` , this is because the string value is wrapped inside an `Optional` enum ( ie. Optional enum `case some(Wrapped)`, "Cupnoodle" is the "Wrapped" variable).

<span id="force-unwrap"></span>
### Force unwrap
To get the wrapped value inside Optional, the easiest way is to use `!`, also known as **force unwrap**.

```swift
let middleName : String? = "Cupnoodle"

// force unwrap it
print("\(middleName!)")
// this will print 'Cupnoodle' instead of 'Optional("Cupnoodle")'
```

<br>

Exclaimation mark (!) is used for forced unwrap to indicate danger, danger as in your app **will crash** if the unwrapped variable is a nil.

If you force unwrap a nil optional variable, your app will crash with error like this:  
![Force unwrap error](https://iosimage.s3.amazonaws.com/2018/20-optionals/forcedUnwrapError.png)
(fatal error: unexpectedly found nil while unwrapping an Optional value)

Only perform force unwrap if you are absolutely sure that the variable is not nil.

<span id="optional-binding"></span>
### Optional binding

What if you are unsure whether the variable is nil but you still want to use it?

A simple way to do it is to check if it is nil before force unwrapping:  
```swift
let middleName: String?  = nil

if middleName != nil {
    print("\(middleName!)")
}

// won't crash as force unwrapping won't occur if middleName is nil
```

<br>

We can assign a non-optional variable to hold the unwrapped value as well:  

```swift
let middleName: String?  = "Cupnoodle"

if middleName != nil {
    // non optional String to store unwrapped value
    let unwrappedMiddleName : String = middleName!
    print("\(unwrappedMiddleName)")
}

// output 'Cupnoodle'
```

<br>

Since the pattern of previous example (assigning a non-optional variable to hold the unwrapped value) is commonly used, Swift has a shortcut for it:  

```swift
let middleName: String?  = "Cupnoodle"

// non optional String to store unwrapped value
if let unwrappedMiddleName = middleName {
    print("\(unwrappedMiddleName)")
}

// output 'Cupnoodle'
```

<br>

This is called **optional binding** .

Swift also allow you to use back the same variable name for optional binding, if the same variable name is used, the non-optional / unwrapped variable will be used inside the `if let a = a { ... }` or `if var a = a { ... }` scope.


```swift
let middleName: String?  = "Cupnoodle"

// non optional String to store unwrapped value
// notice the same variable name 'middleName' on left and right
if let middleName = middleName {
    // the middleName used inside the 'if let' is non-optional / unwrapped
    print("\(middleName)")
}

// output 'Cupnoodle'
```

<br>

Swift also allow you to do optional binding for multiple variable in one go like this:  
```swift
let firstName: String? = "Axel"
let lastName: String? = "Kee"

// only perform the code inside if both firstName and lastName is not nil
if let firstName = firstName,
   let lastName = lastName {
   
   print("\(firstName) \(lastName)")
}
```

<br>

Optional binding first check if the variables are nil or not, if all the variables are not nil, then it will take the unwrapped values and execute the code inside it.

<span id="optional-chaining"></span>
### Optional chaining
Checking for nil value in optional can be tedious at times. There might be scenario where you only want to perform a function on an object when the object isn't nil. 

For example, let's say we have a 'eat' function in `Person` struct and we want the app to call 'eat' function only when the person is not nil.

```swift
struct Person {
    let firstName: String
    let middleName: String?
    let lastName: String
    
    func eat(){
        print("\(firstName) nom nom")
    }
}
```

<br>

Well we can do this with optional binding :  
```swift
let somebody : Person? = nil

if let somebody = somebody {
    somebody.eat()
}
```

<br>

Or in even simpler form, **optional chaining** :  
```swift
let somebody : Person? = nil

// eat() will only be called if somebody is not nil
somebody?.eat()
```

<br>

You can put a question mark `?` after an optional variable (eg: somebody?), this will make the code check if the variable is nil or not. If it is not nil, the code execution will continue to the right (in this case, the 'eat()' function). If it is nil, the execution on current line will stop and the next line will be executed.

Example:  

![Asriel nom](https://iosimage.s3.amazonaws.com/2018/20-optionals/asrielNom.png)

You can keep on chaining optionals, the chain will stop execution if any one of the optionals in it is nil, it will only execute if all of the optionals are not nil.

Example below shows that `makeNoise()` will only be called if Person is not nil and the person's pet is not nil.

```swift
struct Person {
    let firstName: String
    let middleName: String?
    let lastName: String
    let pet: Pet?
	
    func eat(){
        print("\(firstName) nom nom")
    }
}

struct Pet {
    func makeNoise(){
        print("woof")
    }
}

let vulpes : Pet? = Pet()
let axel : Person? = Person(firstName: "Axel", middleName: nil, lastName: "Kee", pet: vulpes)

// makeNoise will only execute if axel is not nil and axel's pet is not nil
axel?.pet?.makeNoise()

//output 'woof'
```

<br>

<span id="nil-coalescing"></span>
### Nil coalescing
Sometimes you will want a default value to be used if the optional variable is nil, for example:  

```swift
// Mcdonalds
var sides : String? = nil

if let sides = sides{
    print("Meal with \(sides)")
} else {
    // if no sides is specified (nil), default sides is french fries
    print("Meal with French Fries")
}

// output 'Meal with French Fries'
```

<br>

We can further simplify this to :

```swift
// Mcdonalds
var sides : String? = nil

// if sides is nil, French Fries will be used
let confirmedSides : String = sides ?? "French Fries"
print("Meal with \(confirmedSides)")

// output 'Meal with French Fries'
```

<br>

The double question mark `??` means that the value after it ("French Fries") will be used if the value before it ( sides ) is nil.

The double question mark `??` is the **nil coalescing operator**.



<span id="implicit"></span>

## Implicitly unwrapped optional

Sometimes you might see a variable type with an exclaimation mark "!" behind it like this :

```swift
@IBOutlet weak var priceLabel: UILabel!
```

<br>

When you link an UI from storyboard to view controller, Xcode will create a type eg: "UILabel" and an exclaimation mark behind the type. What does this mean?



It is similar to **var priceLabel: UILabel ?** . When we define an optional using **?** (eg: weak var name: String?) and want to use it, we have to either unwrap it using **!** (name!) or use optional binding **if let name = name** etc.



This can become tedious when you need to use this variable often and you know this variable is never going to be **nil**. To save some keystroke, we can declare the variable to be like this:

```swift
weak var userProfileURLString: String! 

/* This will declare userProfileURLString as optional, but it will auto use 'userProfileURLString!' when you call 'userProfileURLString' like this 
let url = URL(string: userProfileURL)
*/
```

<br>



In the above example, **userProfileURLString** is an optional variable, meaning it can store nil. But when you access it in code, it will auto call "**!**" to unwrap itself even if you didn't put exclaimation mark behind it.



```swift
weak var optionalString: String?
weak var implicitString: String!

// need to use '!' manually
let url1 = URL(string: optionalString!)

// Swift will auto use '!' for us, so we wont need to type '!' manually
let url2 = URL(string: implicitString)

// if we change it to nil
implicitString = nil
let url3 = URL(string: implicitString)
// this will crash the app, because Swift will auto use '!' to force unwrap it
```

<br>



The use case for this is when you are absolutely sure the variable won't be nil (eg: when you link the UI in storyboard, the UI always exist in the storyboard, hence it will not be nil).



**var implicitString: String!** is an implicitly unwrapped optional.



<span id="summary"></span>

## Summary
I hope you have gained more understanding about Optional and various jargons around it after reading this post.

We can't avoid the existence of Nil in code, there will always be nil values returned (eg: middleName, or when there is no user exist for a particular userID etc). Optional makes dealing with nil easier and it warns most of the error about Nil at compile time so we can fix it before deploying the app! (Better than having user experiencing crash when using the app)


<span id="guard"></span>
## Simplify optionals handling with guard
You might heard of or seen the keyword `guard` before, `guard` is introduced in Swift 2.0 and it does simplify scenario where you need to handle multiple nil optionals differently.

To keep this article from going on too long, I have condensed the explanation of **how and when to use guard** to handle optionals into an email. If you are interested, submit the form below and I will send you the explanation straight to your inbox.

<div class="post-subscribe" style="margin-top:0;">
  <div class="post-subscribe-left">
      <h4> Get the explanation of how and when to use <strong>guard</strong></h4>
    <span style="margin-left:auto; margin-right:auto;"> 
            <img src="https://iosimage.s3.amazonaws.com/markdown.png"></img>
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/969994431/submissions" method="post" data-drip-embedded-form="969994431">
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
                <input type="submit" value="Send me the explanation!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Weekly ish iOS Development tips to make you a better iOS Developer. <br> Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>