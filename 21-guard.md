# Simplify optional handling using Guard

In the [optional and jargons post](fluffy.es/eli-5-optional/) , we have discussed about using `if let` (optional binding) to handle optional.

`if let` will check if the variables are nil and only execute the block if it is not nil. Other than that, the variable declared in `if let` is only available **inside** the block, meaning you can't use the unwrapped variable outside of `if let`:  

```swift
// Mcdonalds
var sides : String? = "Corn"

if let unwrappedSides = sides {
    print("sides is \(unwrappedSides)")
}

// can't use unwrappedSides outside of 'if let'
print("sides is \(unwrappedSides)")  
// error "Use of unresolved identifier 'unwrappedSides'"
```

<br>

Even if we use back the same variable name in `if let`, we still get the optional / wrapped variable outside of the block :

```swift
var sides : String? = "Corn"

if let sides = sides {
    print("sides is \(sides)")
}

print("sides is \(sides)")  
// output Optional("Corn")
```

## Handling optional with if let

Let's say we have a registration screen with three fields like this:  
![Login Screen](https://iosimage.s3.amazonaws.com/2018/21-guard/loginScreen.png)

We want to send the username, email and password to a web REST API only if three of them are filled. We can do it like this : 

```swift
// textfield's text property is optional String type

// only call API if all of the field is filled with 1 or more characters
if let username = usernameTextField.text, 
   let email = emailTextField.text,
   let password = passwordTextField.text,
   username.count > 0,
   email.count > 0,
   password.count > 0 {
   
   callRegisterAPI(username: username, email: email, password: password)
}
   
```

<br>

Say we want to show an alert when user left one of the field blank, using `if let`, we can do it like this:  
```swift
if let username = usernameTextField.text,
   username.count > 0 {
   // ...
} else {
    showAlert(msg: "Username cannot be blank")
    return // stop execution, code below won't get excuted
}

if let email = emailTextField.text,
   email.count > 0 {
   // ...
} else {
    showAlert(msg: "Email cannot be blank")
    return // stop execution, code below won't get excuted
}

if let password = passwordTextField.text,
   password.count > 0 {
   // ...
} else {
    showAlert(msg: "Password cannot be blank")
    return // stop execution, code below won't get excuted
}

// can't use the username, email and password variable as 
// they are not available outside of their `if let` scope.

// force unwrap the text from textfield as we know they are not nil at this point
callRegisterAPI(username: usernameTextField.text!, email: emailTextField.text!, password: passwordTextField.text!)
```

<br>

It looks okay but at the end we have to force unwrap the textfield's text (eg: `usernameTextField.text!`) when calling the register API method because the unwrapped `username`, `email` and `password` in `if let` is not available outside their `if let` scope.

With `guard`, the force unwrapped variable can be accessed outside their scope unlike `if let`, we will convert the code to use `guard` in the next section.

## Handling optional with guard let
`guard let` must be immediately followed by an `else`. 
`guard let` will check if the variable is nil and execute the else block  **if** the variable is nil.

```swift
guard let username = usernameTextField.text else {
    // if it is nil
    print("username is nil")
    return
}
```

<br>

The difference between `guard let` and `if let` is that the unwrapped variable can be used **outside** the scope of `guard let`.

```swift
guard let username = usernameTextField.text else {
    // if it is nil
    print("username is nil")
}


// 'username' can be used outside guard let
print("username is \(username)")
// outputs username, no error
```

<br>

Using the login screen example, we can simplify the code to this:  

```swift
// check if username is empty, if it is, show alert and return (stop executing line below)
guard let username = usernameTextField.text,
   username.count > 0  else {
    showAlert(msg: "Username cannot be blank")
    return
}

// check if email is empty, if it is, show alert and return (stop executing line below)
guard let email = emailTextField.text,
   email.count > 0 else {
    showAlert(msg: "Email cannot be blank")
    return
}

// check if password is empty, if it is, show alert and return (stop executing line below)
guard let password = passwordTextField.text,
   password.count > 0 else {
    showAlert(msg: "Password cannot be blank")
    return // stop execution, code below won't get excuted
}

// We can use the unwrapped variable outside the 'guard let' scope!
callRegisterAPI(username: username, email: email, password: password)
```

<br>


With `guard`, we can check for nil and handle nil case differently for each optional variable, and we can use the unwrapped variable outside its scope!

Using `guard let` simplifies optional handling as the unwrapped variable can be used outside its scope, the caveat of using guard is that there must be a `return` or `throw` statement inside the `else` block. This is to ensure that the function containing guard statement will end execution if there is any nil variables.

If we remove `return` inside the `else` block, Xcode will shows us an error like this:  
![No return error](https://iosimage.s3.amazonaws.com/2018/21-guard/noReturn.png)

<br>

`guard let` is useful when we want to handle different error for different optional variables. It also let us code defensively by first checking error and only continue execution if there is no error.