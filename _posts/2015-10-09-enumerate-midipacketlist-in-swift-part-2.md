---
layout: post
---

# Enumerate MIDIPacketList in Swift: Part 2

> This is the conclusion of [Enumerate MIDIPacketList in Swift](http://design.featherless.software/midi-packet-sequences-in-swift-part-1/).

In [Part 1](http://design.featherless.software/midi-packet-sequences-in-swift-part-1/) we examined the structure of MIDIPacketList and built an Objective-C-influenced implementation of SequenceType. In this post we'll look at Swift 2's AnyGenerator type to achieve the same effect &mdash; but more concisely &mdash; thanks to Swift [type inference] and [closures].

## AnyGenerator

Swift 2's [AnyGenerator] type - and corresponding [anyGenerator] method - allow us to create inline generators. This removes the need to create a separate Generator class and &mdash; paired with type inference &mdash; allows us to remove a lot of boilerplate. Let's re-examine our MIDIPacketList extension.

The essential compilable code for a SequenceType built with AnyGenerator looks like this:

```language-swift
extension MIDIPacketList: SequenceType {
  public func generate() -> AnyGenerator<MIDIPacket> {
    // TODO: Local state...

    return anyGenerator({
      // TODO: return nil if no additional elements

      // TODO: Iterator logic

      return nil // TODO: return the next sequence item
    })
  }
}
```

- Swift is able to infer the Generator type based on the return value of `generate()`.
- A Generator returns nil to indicate that it has no more elements to generate.

The Generator in the code above will not generate anything. Let's fill in the implementation.

#### Step 1: Iterator state

First we define the generator's iterator.

```language-swift
public func generate() -> AnyGenerator<MIDIPacket> {
  var iterator: MIDIPacket?
  var nextIndex: UInt32 = 0
```

- `iterator` stores the last packet we returned from the generator. Its initial nil value indicates that we haven't enumerated a packet yet.
- `nextIndex` allows us to know once we've consumed all packets. This is required because MIDIPacketNext advances packet pointers by a fixed amount on each invocation, eventually pointing to memory not allocated for packets. nextPacket alone is not a reliable signal for having consumed all packets.

#### Step 2: Iterator logic

Next we define the logic that will advance the iterator on each call of the generator.

```language-swift
  return anyGenerator {
    if iterator == nil {
      iterator = self.packet
    } else {
      iterator = withUnsafePointer(&iterator!) { MIDIPacketNext($0).memory }
    }
    return iterator
  }
```

- `iterator`'s nil state allows us to return self.packet initially.
- Each subsequent call advances `iterator` with MIDIPacketNext.
- MIDIPacketNext accepts and returns an [UnsafePointer]. [withUnsafePointer] is a helpful way to transform between MIDIPacket and UnsafePointer<MIDIPacket>, avoiding the need to initialize an [UnsafeMutablePointer] ourselves.
- The `$0` is [shorthand for the closure's first argument][closure-shorthand].
- The enclosing `()` for the anyGenerator function has been removed because they're not needed when the only argument is a closure.

#### Step 3: Iterator termination

The final implementation advances and checks `nextIndex` against `self.numPackets` before iterating in order to avoid accessing invalid memory.

```language-swift
public func generate() -> AnyGenerator<MIDIPacket> {
  var iterator: MIDIPacket?
  var nextIndex: UInt32 = 0

  return anyGenerator {
    if nextIndex++ >= self.numPackets { return nil }
    if iterator == nil {
      iterator = self.packet
    } else {
      iterator = withUnsafePointer(&iterator!) { MIDIPacketNext($0).memory }
    }
    return iterator

  }
}
```

## Addendum

This series went through multiple iterations as I learned more Swift language features. Check out [the revision history of this entry's code](https://gist.github.com/jverkoey/defb7f9f3578d5cb3ff3/revisions).

[type inference]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID322
[closures]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html#//apple_ref/doc/uid/TP40014097-CH11-ID94
[anyGenerator]: http://swiftdoc.org/v2.0/func/anyGenerator/
[AnyGenerator]: http://swiftdoc.org/v2.0/type/AnyGenerator/
[withUnsafePointer]: http://swiftdoc.org/v2.0/func/withUnsafePointer/
[closure-shorthand]: https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html#//apple_ref/doc/uid/TP40014097-CH11-ID100
[UnsafeMutablePointer]: http://swiftdoc.org/v2.0/type/UnsafeMutablePointer/
[UnsafePointer]: http://swiftdoc.org/v2.0/type/UnsafePointer/
