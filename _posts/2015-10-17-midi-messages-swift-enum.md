---
layout: post
---

# MIDI messages as a Swift associated enum

The [MIDI specification] outlines the complete set of messages one can expect to receive from a MIDI endpoint. Each message has its own distinct parameters and values. This is a great opportunity to use an [enum type with associated values](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html#//apple_ref/doc/uid/TP40014097-CH12-ID148).

```language-swift
public enum Message {
  case NoteOff(channel: UInt8, key: UInt8, velocity: UInt8)
  case NoteOn(channel: UInt8, key: UInt8, velocity: UInt8)
  case Aftertouch(channel: UInt8, key: UInt8, pressure: UInt8)
  case ControlChange(channel: UInt8, controller: UInt8, value: UInt8)
  case ProgramChange(channel: UInt8, programNumber: UInt8)
  case ChannelPressure(channel: UInt8, pressure: UInt8)
  case PitchBend(channel: UInt8, pitch: UInt16)
}
```

With this enum we can now create a MIDI message like so:

```language-swift
Message.NoteOff(channel: channel, key: key, velocity: velocity)
```

And react to messages in a switch statement like so:

```language-swift
switch message {
case .NoteOn(let channel, let key, let velocity):
  print(velocity)
case .NoteOff(let channel, let key, let velocity):
  print(key)
case .ControlChange(let channel, let controller, let value):
  print(value)
default:
  break
}
```

[MIDI specification]: http://www.midi.org/techspecs/midimessages.php
