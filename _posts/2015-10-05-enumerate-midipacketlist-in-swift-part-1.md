---
layout: post
---

# Enumerate MIDIPacketList in Swift: Part 1

In this post we're going to look at how we can use Swift 2's SequenceType and GeneratorType to enumerate the packets of a [MIDIPacketList].

- A [SequenceType] is "a type that can be iterated with a for...in loop."
- A [GeneratorType] "encapsulates iteration state and interface for iteration over a sequence."

But before we dive into the implementation, let's understand how MIDI packets are represented in Swift.

## What is a packet list?

CoreMIDI transports messages via [MIDIPacketList], which is essentially a linked list of [MIDIPacket] structs in that its packets can only be sequentially iterated over.

```language-swift
// Extracted from the CoreMIDI Swift framework
public struct MIDIPacketList {
    public var numPackets: UInt32
    public var packet: (MIDIPacket)
    public init()
    public init(numPackets: UInt32, packet: (MIDIPacket))
}
```

Notice that `packet` is a singular MIDIPacket, not an array like you might expect. This is a side effect of the Objective-C struct from which this Swift struct was generated:

```language-objectivec
// Extracted from the CoreMIDI Objective-C framework header
struct MIDIPacketList
{
	UInt32  			numPackets;	
	MIDIPacket  		packet[1];
};
```

So how do we iterate over the packets? The answer was introduced in Swift 2: [MIDIPacketNext].

> **MIDIPacketNext** is an Objective-C macro made available in Swift 2. Contrary to what the docs imply, MIDIPacketNext works on pre-iOS 9 devices running Swift thanks to the embedded runtime.

A C implementation might look like this:

```language-objectivec
MIDIPacket *packet = &packetList->packet[0];
for (int i = 0; i < packetList->numPackets; ++i) {
  ...
  packet = MIDIPacketNext(packet);
}
```

But using Swift extensions we're going to allow enumeration like this:

```language-swift
let packetList: MIDIPacketList
for packet in packetList {
  ...
}
```

---

## MIDIPacketList as a Sequence

Treating MIDIPacketList like a sequence is easy with Swift extensions:

```language-swift
extension MIDIPacketList: SequenceType {
}
```

> **Why SequenceType instead of CollectionType?**
>
> CollectionType requires subscript access at **arbitrary** indexes. This isn't needed when processing a stream of MIDI packets. SequenceType is a more accurate model of how MIDIPackets are consumed. [Learn more](http://stackoverflow.com/questions/30565875/what-is-the-difference-between-sequencetype-and-collectiontype-in-swift).

SequenceType requires two things for conformity:

- a `Generator` typealias, and
- an implementation of `func generate() -> Generator`.

### Classic implementation

Someone coming from Objective-C has been trained to see a protocol and mechanically start implementing its required methods. In this case that would entail defining the Generator type:

```language-swift
public typealias Generator = MIDIPacketListGenerator
```

and implementing the generate method:

```language-swift
public func generate() -> Generator {
  return Generator(packetList: self)
}
```

MIDIPacketListGenerator must be a GeneratorType, so we'd implement the struct accordingly:

```language-swift
public struct MIDIPacketListGenerator: GeneratorType {
  public typealias Element = MIDIPacket
  public mutating func next() -> Element? {
    // iterator logic
  }
}
```

With a working implementation looking something like this:

```language-swift
public struct MIDIPacketListGenerator : GeneratorType {
  public typealias Element = MIDIPacket

  init(packetList: MIDIPacketList) {
    let ptr = UnsafeMutablePointer<MIDIPacket>.alloc(1)
    ptr.initialize(packetList.packet)
    self.packet = ptr
    self.count = packetList.numPackets
  }

  public mutating func next() -> Element? {
    guard self.packet != nil && self.index < self.count else { return nil }

    let lastPacket = self.packet!
    self.packet = MIDIPacketNext(self.packet!)
    self.index++
    return lastPacket.memory
  }

  // Extracted packet list info
  var count: UInt32
  var index: UInt32 = 0

  // Iteration state
  var packet: UnsafeMutablePointer<MIDIPacket>?
}
```

But this is a pretty long solution in Swift, a language with type inference and a variety of other useful tools. In part 2 of this post we'll reduce the size of this implementation using Swift's [AnyGenerator] type. [Read Part 2 now](http://design.featherless.software/enumerate-midipacketlist-in-swift-part-2/).

[View the full implementation of this approach](https://gist.github.com/jverkoey/defb7f9f3578d5cb3ff3/2055e0ec9f854b4511e799d15411886242e68ea5).

## References

- [Swift 2 and CoreMIDI](http://www.rockhoppertech.com/blog/swift-2-and-coremidi/) by Gene De Lisa

[MIDIPacketList]: https://developer.apple.com/library/ios/documentation/CoreMidi/Reference/MIDIServices_Reference/#//apple_ref/c/tdef/MIDIPacketList
[MIDIPacket]: https://developer.apple.com/library/prerelease/ios/documentation/CoreMidi/Reference/MIDIServices_Reference/index.html#//apple_ref/c/tdef/MIDIPacket
[SequenceType]: http://swiftdoc.org/v2.0/protocol/SequenceType/
[GeneratorType]: http://swiftdoc.org/v2.0/protocol/GeneratorType/
[MIDIPacketNext]: https://developer.apple.com/library/ios/documentation/CoreMidi/Reference/MIDIServices_Reference/#//apple_ref/c/func/MIDIPacketNext
[AnyGenerator]: http://swiftdoc.org/v2.0/type/AnyGenerator/
