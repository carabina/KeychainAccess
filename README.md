# KeychainAccess
[![CI Status](http://img.shields.io/travis/kishikawakatsumi/KeychainAccess.svg?style=flat)](https://travis-ci.org/kishikawakatsumi/KeychainAccess)
[![Carthage Compatibility](https://img.shields.io/badge/carthage-✓-f2a77e.svg?style=flat)](https://github.com/Carthage/Carthage/)
[![Version](https://img.shields.io/cocoapods/v/KeychainAccess.svg?style=flat)](http://cocoadocs.org/docsets/KeychainAccess)
[![License](https://img.shields.io/cocoapods/l/KeychainAccess.svg?style=flat)](http://cocoadocs.org/docsets/KeychainAccess)
[![Platform](https://img.shields.io/cocoapods/p/KeychainAccess.svg?style=flat)](http://cocoadocs.org/docsets/KeychainAccess)

KeychainAccess is a simple Swift wrapper for Keychain that works on iOS and OS X. Makes using Keychain APIs exremely easy and much more palatable to use in Swift.

<img src="https://raw.githubusercontent.com/kishikawakatsumi/KeychainAccess/master/Screenshots/01.png" width="320px" />

## :bulb: Features

- Simple interface
- Support access group
- [Support accessibility](#accessibility)
- [Support iCloud sharing](#icloud_sharing)
- **[Support TouchID and Keychain integration (iOS 8+)](#touch_id_integration)**
- Works on both iOS & OS X

## :book: Usage

##### :eyes: See also:  
- [:link: Playground](https://github.com/kishikawakatsumi/KeychainAccess/blob/master/Examples/Playground-iOS.playground/section-1.swift)  
- [:link: iOS Example Project](https://github.com/kishikawakatsumi/KeychainAccess/tree/master/Examples/Example-iOS)

### :key: Basics

#### Saving Application Password

```swift
let keychain = Keychain(service: "com.example.github-token")
keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

#### Saving Internet Password

```swift
let keychain = Keychain(server: NSURL(string: "https://github.com")!, protocolType: .HTTPS)
keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

### :key: Instantiation

#### Create Keychain for Application Password

```swift
let keychain = Keychain(service: "com.example.github-token")
```

```swift
let keychain = Keychain(service: "com.example.github-token", accessGroup: "12ABCD3E4F.shared")
```

#### Create Keychain for Internet Password

```swift
let keychain = Keychain(server: NSURL(string: "https://github.com")!, protocolType: .HTTPS)
```

```swift
let keychain = Keychain(server: NSURL(string: "https://github.com")!, protocolType: .HTTPS, authenticationType: .HTMLForm)
```

### :key: Adding an item

#### subscripting

```swift
keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

#### set method

```swift
keychain.set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
```

#### error handling

```swift
if let error = keychain.set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi") {
    println("error: \(error)")
}
```

### :key: Obtaining an item

#### subscripting (automatically converts to string)

```swift
let token = keychain["kishikawakatsumi"]
```

#### get methods

##### as String

```swift
let token = keychain.get("kishikawakatsumi")
```

```swift
let token = keychain.getString("kishikawakatsumi")
```

##### as NSData

```swift
let data = keychain.getData("kishikawakatsumi")
```

#### error handling

**First, get the `failable` (value or error) object**

```swift
let failable = keychain.getStringOrError("kishikawakatsumi")
```

**1. check `enum` state**

```swift
switch failable {
case .Success:
  println("token: \(failable.value)")
case .Failure:
  println("error: \(failable.error)")
}
```

**2. check `error` object**

```swift
if let error = failable.error {
    println("error: \(failable.error)")
} else {
    println("token: \(failable.value)")
}
```

**3. check `failed` property**

```swift
if failable.failed {
    println("error: \(failable.error)")
} else {
    println("token: \(failable.value)")
}
```

### :key: Removing an item

#### subscripting

```swift
keychain["kishikawakatsumi"] = nil
```

#### remove method

```swift
keychain.remove("kishikawakatsumi")
```

#### error handling

```swift
if let error = keychain.remove("kishikawakatsumi") {
    println("error: \(error)")
}
```

### :key: Label and Comment

```swift
let keychain = Keychain(server: NSURL(string: "https://github.com")!, protocolType: .HTTPS)
keychain
    .label("github.com (kishikawakatsumi)")
    .comment("github access token")
    .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
```

### :key: Configuration (Accessibility, Sharing, iCould Sync)

**Provides fluent interfaces**

```swift
let keychain = Keychain(service: "com.example.github-token")
    .label("github.com (kishikawakatsumi)")
    .synchronizable(true)
    .accessibility(.AfterFirstUnlock)
```

#### <a name="accessibility"> Accessibility

##### Default accessibility matches background application (=kSecAttrAccessibleAfterFirstUnlock)

```swift
let keychain = Keychain(service: "com.example.github-token")
```

##### For background application

###### Creating instance

```swift
let keychain = Keychain(service: "com.example.github-token")
    .accessibility(.AfterFirstUnlock)

keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

###### One-shot

```swift
let keychain = Keychain(service: "com.example.github-token")

keychain
    .accessibility(.AfterFirstUnlock)
    .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
```

##### For foreground application

###### Creating instance

```swift
let keychain = Keychain(service: "com.example.github-token")
    .accessibility(.WhenUnlocked)

keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

###### One-shot

```swift
let keychain = Keychain(service: "com.example.github-token")

keychain
    .accessibility(.WhenUnlocked)
    .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
```

#### :couple: Sharing Keychain items

```swift
let keychain = Keychain(service: "com.example.github-token", accessGroup: "12ABCD3E4F.shared")
```

#### <a name="icloud_sharing"> :arrows_counterclockwise: Synchronizing Keychain items with iCloud

###### Creating instance

```swift
let keychain = Keychain(service: "com.example.github-token")
    .synchronizable(true)

keychain["kishikawakatsumi"] = "01234567-89ab-cdef-0123-456789abcdef"
```

###### One-shot

```swift
let keychain = Keychain(service: "com.example.github-token")

keychain
    .synchronizable(true)
    .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")
```

### <a name="touch_id_integration"> :fu: Touch ID integration

**Any Operation that require authentication must be run in the background thread.**  
**If you run in the main thread, UI thread will lock for the system to try to display the authentication dialog.**

#### :closed_lock_with_key: Adding a Touch ID protected item

If you want to store the Touch ID protected Keychain item, specify `accessibility` and `authenticationPolicy` attributes.  

```swift
let keychain = Keychain(service: "com.example.github-token")

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
    let error = keychain
        .accessibility(.WhenPasscodeSetThisDeviceOnly, authenticationPolicy: .UserPresence)
        .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")

    if error != nil {
        // Error handling if needed...
    }
}
```

#### :closed_lock_with_key: Updating a Touch ID protected item

The same way as when adding.  

**Do not run in the main thread if there is a possibility that the item you are trying to add already exists, and protected.**
**Because updating protected items requires authentication.**

Additionally, you want to show custom authentication prompt message when updating, specify an `authenticationPrompt` attribute.
If the item not protected, the `authenticationPrompt` parameter just be ignored.

```swift
let keychain = Keychain(service: "com.example.github-token")

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
    let error = keychain
        .accessibility(.WhenPasscodeSetThisDeviceOnly, authenticationPolicy: .UserPresence)
        .authenticationPrompt("Authenticate to update your access token")
        .set("01234567-89ab-cdef-0123-456789abcdef", key: "kishikawakatsumi")

    if error != nil {
        // Error handling if needed...
    }
}
```

#### :closed_lock_with_key: Obtaining a Touch ID protected item

The same way as when you get a normal item. It will be displayed automatically Touch ID or passcode authentication If the item you try to get is protected.  
If you want to show custom authentication prompt message, specify an `authenticationPrompt` attribute.
If the item not protected, the `authenticationPrompt` parameter just be ignored.

```swift
let keychain = Keychain(service: "com.example.github-token")

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
    let failable = keychain
        .authenticationPrompt("Authenticate to login to server")
        .getStringOrError("kishikawakatsumi")

    if failable.successed {
        println("value: \(failable.value)")
    } else {
        println("error: \(failable.error?.localizedDescription)")
        // Error handling if needed...
    }
}
```

#### :closed_lock_with_key: Removing a Touch ID protected item

The same way as when you remove a normal item.
There is no way to show Touch ID or passcode authentication when removing Keychain items.

```swift
let keychain = Keychain(service: "com.example.github-token")

let error = keychain.remove("kishikawakatsumi")

if error != nil {
    println("error: \(error?.localizedDescription)")
    // Error handling if needed...
}
```

### :key: Debugging

#### Display all stored items if print keychain object

```swift
let keychain = Keychain(server: NSURL(string: "https://github.com")!, protocolType: .HTTPS)
println("\(keychain)")
```

```
=>
[
  [authenticationType: Default, key: kishikawakatsumi, server: github.com, class: InternetPassword, protocol: HTTPS]
  [authenticationType: Default, key: hirohamada, server: github.com, class: InternetPassword, protocol: HTTPS]
  [authenticationType: Default, key: honeylemon, server: github.com, class: InternetPassword, protocol: HTTPS]
]
```

#### Obtaining all stored keys

```swift
let keychain = Keychain(server: NSURL(string: "https://github.com")!, protocolType: .HTTPS)

let keys = keychain.allKeys()
for key in keys {
  println("key: \(key)")
}
```

```
=>
key: kishikawakatsumi
key: hirohamada
key: honeylemon
```

#### Obtaining all stored items

```swift
let keychain = Keychain(server: NSURL(string: "https://github.com")!, protocolType: .HTTPS)

let items = keychain.allItems()
for item in items {
  println("item: \(item)")
}
```

```
=>
item: [authenticationType: Default, key: kishikawakatsumi, server: github.com, class: InternetPassword, protocol: HTTPS]
item: [authenticationType: Default, key: hirohamada, server: github.com, class: InternetPassword, protocol: HTTPS]
item: [authenticationType: Default, key: honeylemon, server: github.com, class: InternetPassword, protocol: HTTPS]
```

## Requirements

iOS 7 or later
OS X 10.9 or later

## Installation

### CocoaPods

KeychainAccess is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

`pod 'KeychainAccess'`

*Currently, to install the framework via CocoaPods you need to use pre-release version.*  
See ["Pod Authors Guide to CocoaPods Frameworks"](http://blog.cocoapods.org/Pod-Authors-Guide-to-CocoaPods-Frameworks/)

### Carthage

KeychainAccess is available through [Carthage](https://github.com/Carthage/Carthage). To install
it, simply add the following line to your Cartfile:

`github "kishikawakatsumi/KeychainAccess"`

### To manually add to your project

1. Add `Lib/KeychainAccess.xcodeproj` to your project
2. Link `KeychainAccess.framework` with your target
3. Add `Copy Files Build Phase` to include the framework to your application bundle

_See [iOS Example Project](https://github.com/kishikawakatsumi/KeychainAccess/tree/master/Examples/Example-iOS) as reference._

<img src="https://raw.githubusercontent.com/kishikawakatsumi/KeychainAccess/master/Screenshots/Installation.png" width="800px" />

## Author

kishikawa katsumi, kishikawakatsumi@mac.com

## License

KeychainAccess is available under the MIT license. See the LICENSE file for more info.
