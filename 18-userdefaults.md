# Save custom object / type into UserDefaults using Codable



If you have tried to save custom object into UserDefaults before, you might get an error like this: 
`'NSInvalidArgumentException', reason: 'Attempt to insert non-property list object appName.Player(name: "Axel", highScore: 42) for key current_player'`

According to Apple's [documentation on UserDefaults](https://developer.apple.com/documentation/foundation/userdefaults#1664798),

> A default object must be a property list—that is, an instance of (or for collections, a combination of instances of) NSData, NSString, NSNumber, NSDate, NSArray, or NSDictionary.

As UserDefaults can only accept those types mentioned above, to save a custom object into UserDefault, we have to convert it into NSData first.

Before Apple introduced the Codable protocol, the way to convert custom object into NSData is using NSKeyedArchiver with NSCoding protocol like this:  
```swift
// only class can be conformed to NSCoding, struct can't.
// class must inherit from NSObject to use NSKeyedArchiver.
class Player : NSObject, NSCoding {
	
    var name: String
    var highScore: Int
	
    // Normal initializer
    init(name: String, highScore: Int) {
        self.name = name
        self.highScore = highScore
    }

	
    // MARK: NSCoding
    required convenience init?(coder decoder: NSCoder) {
        guard let name = decoder.decodeObject(forKey: "name") as? String
        else { return nil }
		
        self.init(
            name: name,
            highScore: decoder.decodeInteger(forKey: "highScore")
        )
    }
	
    func encode(with aCoder: NSCoder) {
        aCoder.encode(self.name, forKey: "name")
        aCoder.encode(self.highScore, forKey: "highScore")
    }
}

// ViewController.h
@IBAction func saveUserDefaults(_ sender: Any) {
    let player = Player(name: "Axel", highScore: 42)
    let defaults = UserDefaults.standard
    
    // use NSKeyedArchiver to convert Player object into Data / NSData type
    let playerData = NSKeyedArchiver.archivedData(withRootObject: player)
    defaults.set(playerData, forKey: "player")
}

@IBAction func loadUserDefaults(_ sender: Any) {
    let defaults = UserDefaults.standard
    guard let playerData = defaults.object(forKey: "player") as? Data else {
        return
    }
	
    // Use NSKeyedUnarchiver to convert Data / NSData back to Player object
    guard let player = NSKeyedUnarchiver.unarchiveObject(with: playerData) as? Player else {
        return
    }
		
    print("player name is \(player.name)")
}
```

<br>

The problem with this approach is that there's quite some boilerplate code to write (eg: `required convenience init?(coder decoder: NSCoder)` and `func encode`), and this gets repetitive if the object has a lot of properties. This approach is also prone to mistake as you might mistakenly typed wrong the key string for `forKey` and then scratching your head afterwards figuring why it doesn't work when saving / loading (happened to me twice 😅).

A better way to save custom object into UserDefaults is using the Codable protocol introduced by Apple in Swift 4.

## Using Codable

Using Codable, we can use struct instead of class as it doesn't need to conform to NSObject anymore.

The refactored Player struct would like this:
```swift
struct Player : Codable {
	
    var name: String
    var highScore: Int
}
```

<br>

Surprisingly short huh? You just need to add the `Codable` protocol to it.
Codable is the combination of `Encodable` and `Decodable` protocol, conforming to Codable means conforming to both Encodable and Decodable.
> typealias Codable = Decodable & Encodable

Encodable allows object to be encoded to a certain format using some encoder like JSONEncoder, PropertyListEncoder etc.

Decodable allows object to be decoded from a certain format using some decoder like JSONDecoder, PropertyListDecoder etc.

As to saving / loading the object to / from UserDefaults, we can do it like this:  

```swift
// ViewController.swift

@IBAction func saveUserDefaults(_ sender: Any) {
    let player = Player(name: "Axel", highScore: 42)
    let defaults = UserDefaults.standard
    
    // Use PropertyListEncoder to convert Player into Data / NSData
    defaults.set(try? PropertyListEncoder().encode(player), forKey: "player")
}
	
@IBAction func loadUserDefaults(_ sender: Any) {
    let defaults = UserDefaults.standard
    guard let playerData = defaults.object(forKey: "player") as? Data else {
        return
    }
	
    // Use PropertyListDecoder to convert Data into Player
    guard let player = try? PropertyListDecoder().decode(Player.self, from: playerData) else {
        return
    }
		
    print("player name is \(player.name)")
}
```

<br>

We will now explain what does `try? PropertyListEncoder().encode(player)` do.
`PropertyListEncoder()` will create an instance of PropertyListEncoder, then it will encode the custom struct into Data type and return the converted Data type using `encode()`. 

There's a `try?` in front as the `encode()` method might throw an error if the encoding process fail (eg: might be due to invalid codable structure, similar like parsing an invalid JSON). If the `encode()` throws an error, `try?` will catch the error, and make the whole statement `PropertyListEncoder().encode(player)` return nil. 

If error happens, `try? PropertyListEncoder().encode(player)` will become `nil` and you will end up saving `nil` value for the key `player`. This is fine as by default, UserDefaults' keys that don't have value stored will return nil when loaded.

Similarly, `try? PropertyListDecoder().decode(Player.self, from: playerData)` will attempt to decode the Data saved in UserDefaults and convert it into Player struct then return it. `PropertyListDecoder()` will create an instance of PropertyListDecoder and `decode(Player.self, from: playerData)` will decode the Data from UserDefaults to Player struct (`Player.self` tells the decoder to decode into Player struct type)

There's also a 'try?' in front of `PropertyListDecoder().decode` as the `decode()` method might throw an error as well if the decoding process fail (eg: might be due to invalid codable structure, similar like parsing an invalid JSON).

## Why PropertyListEncoder / PropertyListDecoder is used for UserDefaults?

You might ask, why is it called "PropertyListEncoder" when you are encoding the object to save in UserDefaults, shouldn't it be called "UserDefaultsEncoder" or something?

As the explanation of the inner working of UserDefaults is quite lengthy and to keep this article from going on too long, I have condensed the explanation of how UserDefaults works into an email. If you are interested, submit the form below and I will send you the explanation straight to your inbox.

<div class="post-subscribe" style="margin-top:0;">
  <div class="post-subscribe-left">
    <h4> Get the explanation of how UserDefaults works</h4>
    <span style="margin-left:auto; margin-right:auto;"> 
            <img src="https://iosimage.s3.amazonaws.com/markdown.png"></img>
            </span>
</div>
        <div class="post-subscribe-right">
            <form action="https://www.getdrip.com/forms/877923333/submissions" method="post" data-drip-embedded-form="877923333">
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
                <input type="submit" value="Send me the email!" data-drip-attribute="sign-up-button" />
                <br>
                <span style="font-size: 0.8rem;">+ Bi-weekly ish iOS Development tips. Unsubscribe any time.</span>
              </div>
            </form>
        </div>
    </div>
​    