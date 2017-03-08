---
layout: post
---

# Enumerate messages of a MIDIPacket using Swift reflection

In *[Enumerate MIDIPacketList in Swift](http://design.featherless.software/enumerate-midipacketlist-in-swift-part-2/)* we built an extension for MIDIPacketList that enabled easy enumeration of its MIDIPacket objects. In this post we go one step further by enumerating the messages contained within a MIDIPacket.

## What is a MIDIPacket?

A [MIDIPacket] is a representation of one or more MIDI messages. Most messages consist of a **status byte** followed by one or two **data bytes**.

> [Learn more about the MIDI specification](http://www.midi.org/techspecs/midimessages.php).

Consider the following representation of a MIDIPacket:

![MIDIPacket](/content/images/2015/10/midipacket--1-.svg)

The packet has a length of 10 bytes and contains four distinct MIDI messages.

Given the above example of a packet, Swift's MIDIPacket definition is intriguing:

```language-swift
// Extracted from the CoreMIDI Swift framework
public struct MIDIPacket {
    public var timeStamp: MIDITimeStamp
    public var length: UInt16
    public var data: (UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8)
    public init()
    public init(timeStamp: MIDITimeStamp, length: UInt16, data: (UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8, UInt8))
}
```

In essence MIDIPacket is represented by:

- the length of the packet,
- a tuple of 256 UInt8 values (shortened in the code snippet above).

The bytes aren't represented as an array like one might expect. Instead **the bytes are stored in a fixed-size tuple**. A glance at the Swift header docs reveals:

> [data] is declared to be 256 bytes in length so clients don't have to create custom data structures in simple situations.

Representing the bytes as a tuple ironically makes iterating over the messages *harder* because it's not easy to iterate over tuple values.

We'll use the helper function `generatorForTuple` defined in *[Enumerating tuple values in Swift]* in order to iterate over `data`'s bytes. [Read the post](/enumerating-tuple-values-swift/) to learn more about how this function works.

## Extracting messages

We'll start by making MIDIPacket conform to SequenceType:

```language-swift
extension MIDIPacket: SequenceType {
  public func generate() -> AnyGenerator<Message> {

    return anyGenerator {
      return nil
    }
  }
}
```

- `Message` is a type defined in a separate article, *[MIDI messages as a Swift enum]*.

### State

The generator state includes a **generator** for our tuple values and an **index** that tracks our iteration progress.

```language-swift
  public func generate() -> AnyGenerator<Message> {
    var generator = generatorForTuple(self.data)
    var index: UInt16 = 0
```

- Read *[Enumerating tuple values in Swift]* to learn how `generatorForTuple` works.

### Iterating the generator

We'll design the iterator such that each byte is accessed only once. To do this we'll define an inline closure called `pop` that generates the next `UInt8` value from the data tuple. This closure will also increment our index so that we know how far along we are.

```language-swift
    return anyGenerator {
      func pop() -> UInt8 {
        assert(index < self.length)
        index++
        return generator.next() as! UInt8
      }

      let byte = pop()

      // TODO: Create the Message.

      assert(false, "Unimplemented message \(byte)")
      return nil
    }
```

- We assume that MIDIPacket only contains well-formed messages but still assert in pop as a safeguard against reading past the packet's length.
- `generatorForTuple`'s enumeration value is an `Any` type so we must cast to `UInt8`.

### Creating the message

The type of a MIDI byte is defined by the most-significant bit. 1 is a status byte, 0 is a data byte. This means that data bytes can only provide 7 bits of information.

> For the purposes of this article we're going to assume that the MIDI data is well-formed. In a production environment care must be taken to follow the MIDI specification's recommendation to gracefully ignore malformed messages.

```language-swift
      if (byte & 0x80) == 0x80 { // Status byte
      }
```

If the byte is a status byte then we know that the top four bits represent the **message** and the lower four bits represent the **channel**:

```language-swift
      if (byte & 0x80) == 0x80 { // Status byte
        let status = byte & 0xF0
        let channel = byte & 0x0F
```

We use `message` to identify the MIDI message and the number of data bytes as defined by [the MIDI specification](http://www.midi.org/techspecs/midimessages.php).

```language-swift
      if (byte & 0x80) == 0x80 { // Status byte
        let message = byte & 0xF0
        let channel = byte & 0x0F
        switch message {
        case 0x80: return .NoteOff(channel: channel, key: pop(), velocity: pop())
        case 0x90: return .NoteOn(channel: channel, key: pop(), velocity: pop())
        case 0xA0: return .Aftertouch(channel: channel, key: pop(), pressure: pop())
        case 0xB0: return .ControlChange(channel: channel, controller: pop(), value: pop())
        case 0xC0: return .ProgramChange(channel: channel, programNumber: pop())
        case 0xD0: return .ChannelPressure(channel: channel, pressure: pop())
        case 0xE0:
          // From http://www.midi.org/techspecs/Messages.php
          // The pitch bender is measured by a fourteen bit value. The first data byte contains the
          // least significant 7 bits. The second data bytes contains the most significant 7 bits.
          let low = UInt16(pop() & 0x7F)
          let high = UInt16(pop() & 0x7F)
          return .PitchBend(channel: channel, pitch: (high << 7) | low)
        default:
          assert(false, "Unimplemented message \(byte)")
          return nil
        }
```

- Each invocation of `pop` consumes a single byte from the `data` tuple.
- Pitch bending is a 14 bit value represented across two data bytes.
- We don't need to prefix each return value with `Message` because Swift is able to infer this from our `generate()` method return type.

### Termination case

In order to safely terminate the generator we check `index` against the packet's `length`:

```language-swift
    return anyGenerator {
      if index >= self.length {
        return nil
      }
```

## Concluding thoughts

[View the final implementation on GitHub](https://github.com/jverkoey/swift-midi/blob/master/LUMI/CoreMIDI/MIDIPacket%2BSequenceType.swift).



[Mirror]: https://developer.apple.com/library/ios/documentation/Swift/Reference/Swift_Mirror_Structure/index.html
[MIDIPacket]: https://developer.apple.com/library/prerelease/ios/documentation/CoreMidi/Reference/MIDIServices_Reference/index.html#//apple_ref/c/tdef/MIDIPacket
[Enumerating tuple values in Swift]: /enumerating-tuple-values-swift/
[AnyForwardCollection]: http://swiftdoc.org/v2.0/type/AnyForwardCollection/
[MIDI messages as a Swift enum]: /midi-messages-swift-enum/
