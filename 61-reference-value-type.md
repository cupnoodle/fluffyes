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

Before going into explaining the difference between reference and value type, I think it's crucial to know the concept of Memory Address.



Your computer and iOS devices has a component named RAM (Random Access Memory), which is used to store working data when you are using the device. For example, when you are playing a game, there will be a loading screen to load the graphic assets and audio needed in the level into the memory (RAM), then when you are playing that level, the game will access the graphic / audio / character information from the RAM.



