---
layout: post
---

# Reading MIDI properties in Swift using ambiguous return types

CoreMIDI's interface for accessing MIDI object properties relies on a handful of C get/set methods corresponding to the supported data types: Integer, String, Dictionary, and Data:

- `MIDIObjectGetIntegerProperty`
- `MIDIObjectGetStringProperty`
- `MIDIObjectGetDictionaryProperty`
- `MIDIObjectGetDataProperty`

Calling the wrong function will result in an error of `kMIDIWrongPropertyType`. This means many trips to the header docs in search of the property's type and is all-around not fun.

In this post we'll use Swift's type system and **ambiguous method signatures** to make accessing these properties a bit more pleasant.

## Creating a property reader

We'd normally use an extension to add functionality to a type, but because `MIDIObjectRef` is a typealias for `UInt32` an extension of MIDIObjectRef would also end up extending UInt32.

In order to minimize the property reader's scope let's create a wrapper object, `PropertyReader`. `propertyOf(object)` seems like a nice way to use this API, so let's also create a function and make PropertyReader's initializer `private`.

```language-swift
func propertyOf(objectRef: MIDIObjectRef) -> PropertyReader? {
  if objectRef == 0 {
    return nil
  }
  return PropertyReader(objectRef)
}

class PropertyReader {
  private init(_ objectRef: MIDIObjectRef) {
    self.objectRef = objectRef
  }
  private let objectRef: MIDIObjectRef
}
```

- Note that `propertyOf` may fail to return a property reader if the object ref is 0.

### The public API

CoreMIDI's property types are defined solely in the docs so we'll have to map each property to a type ourselves.

```language-swift
extension PropertyReader {
  var name: String
  var manufacturer: String
  var uniqueID: Int32
  var deviceID: Int32
  ...
```

We only want to call CoreMIDI property methods when needed, so we'll use [computed properties](https://developer.apple.com/library/watchos/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html#//apple_ref/doc/uid/TP40014097-CH14-ID259) to implement the logic that fetches the property. Let's implement `name` and `uniqueID` first:

```language-swift
  var name: String {
    var value: Unmanaged<CFString>?
    let result = MIDIObjectGetStringProperty(self.objectRef, kMIDIPropertyName, &value)
    assert(result == noErr, "Failure with error \(Error(result))")
    return value!.takeRetainedValue() as String
  }

  var uniqueID: Int32 {
    var value: Int32 = 0
    let result = MIDIObjectGetIntegerProperty(self.objectRef, kMIDIPropertyUniqueID, &value)
    assert(result == noErr, "Failure with error \(Error(result))")
    return value
  }
```

This will turn into a lot of boilerplate for each value. Let's extract the common parts into a function for each type:

```language-swift
  private func getString(propertyID: CFString) -> String {
    var value: Unmanaged<CFString>?
    let result = MIDIObjectGetStringProperty(self.objectRef, propertyID, &value)
    assert(result == noErr, "Failure with error \(Error(result))")
    return value!.takeRetainedValue() as String
  }

  private func getInt32(propertyID: CFString) -> String {
    var value: Int32 = 0
    let result = MIDIObjectGetIntegerProperty(self.objectRef, propertyID, &value)
    assert(result == noErr, "Failure with error \(Error(result))")
    return value
  }
```

Our properties now look like this:

```language-swift
  var name: String { return self.getString(kMIDIPropertyName) }
  var manufacturer: String { return self.getString(kMIDIPropertyManufacturer) }
  var uniqueID: Int32 { return self.getInt32(kMIDIPropertyUniqueID) }
  var deviceID: Int32 { return self.getInt32(kMIDIPropertyDeviceID) }
```

This works, but the type is written out twice in each statement: once for the var and once again for the method name.

What if we could have one function that knew which type to return &mdash; and which CoreMIDI function to call &mdash; based on the context? The answer lies in Swift's ability to disambiguate functions.

### Overloading method return types

It turns out that if you have a family of functions that **differ only by return type**, Swift is able to infer which function to call based on the destination variable's type - nifty! From [the Swift docs](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Declarations.html#//apple_ref/doc/uid/TP40014097-CH34-ID379):

> You can overload a subscript declaration in the type in which it is declared, as long as the parameters **or the return type differ** from the one you’re overloading.

Consider the following simple example:

```language-swift
func method() -> Int {
  return 5
}
func method() -> String {
  return "String"
}

let result: String = method()
```

Which `method` is called depends on the type of `result`. Try pasting this code into a Playground and changing `result`'s type to an `Int`.

---

We'll use this to our advantage and build a function called `getValue` for each property type. Notice that the return value is the only difference in each function definition.

```language-swift
  private func getValue(propertyID: CFString) -> String {
    var value: Unmanaged<CFString>?
    let result = MIDIObjectGetStringProperty(self.objectRef, propertyID, &value)
    assert(result == noErr, "Failure with error \(result)")
    return value!.takeRetainedValue() as String
  }

  private func getValue(propertyID: CFString) -> Int32 {
    var value: Int32 = 0
    let result = MIDIObjectGetIntegerProperty(self.objectRef, propertyID, &value)
    assert(result == noErr, "Failure with error \(result)")
    return value
  }
```

Now we can write the following:

```language-swift
  var name: String { return self.getValue(kMIDIPropertyName) }
  var manufacturer: String { return self.getValue(kMIDIPropertyManufacturer) }
  var uniqueID: Int32 { return self.getValue(kMIDIPropertyUniqueID) }
  var deviceID: Int32 { return self.getValue(kMIDIPropertyDeviceID) }
```

and Swift will infer which implementation of `getValue` to call by the type of the expression. **We define the type once and Swift infers the rest for us**.

In [the final implementation on GitHub](https://github.com/jverkoey/swift-midi/blob/master/LUMI/Internal/PropertyReader.swift) we take this one step further and implement `getValue` as a `subscript` function in order to simplify our syntax even further.
