---
layout: post
---

# Enumerating tuple values in Swift

> This post is also available as a Swift playground at https://github.com/jverkoey/playgrounds.

Consider the following:

```language-swift
let fibonacci = (0, 1, 1, 2, 3, 5, 8, 13, 21, 34)
```

How might we iterate over the tuple's values? We might try a for-in loop:

```language-swift
for val in fibonacci {
  val
}
```

But tuples don't conform to SequenceType which the resulting error reiterates:

> Type '(Int, Int, ..., Int)' does not conform to protocol 'SequenceType'

So we might then try to enumerate using a subscript:

```language-swift
fibonacci[0]
```

But tuples don't implement `subscript` so we're met with the following error:

> Type '(Int, Int, ..., Int)' has no subscript members

So how do we iterate over our tuple? The answer is provided by Swift's [Mirror] type. From the docs:

> [Mirror is] a representation of the sub-structure and optional "display style" of any arbitrary subject instance.
>
> Describes the parts---such as stored properties, collection elements, tuple elements, or the active enumeration case---that make up a particular instance.

## Tuple reflection

We create a Mirror by initializing it with the object we intend to reflect.

[Mirror]: https://developer.apple.com/library/ios/documentation/Swift/Reference/Swift_Mirror_Structure/index.html

```language-swift
let mirror = Mirror(reflecting: fibonacci)
```

The resulting mirror object allows us to enumerate through the values of the object's children.

```language-swift
for child in mirror.children {
  child.value
}
```

![Swift Playground output](/content/images/2015/10/Screen-Shot-2015-10-17-at-2-24-21-PM.png)

## Putting it all together

Rather than create a Mirror every time we want to enumerate a tuple, let's build a helper function that turns tuples into enumerable types.

```language-swift
func generatorForTuple(tuple: Any) -> AnyGenerator<Any> {
  return anyGenerator(Mirror(reflecting: tuple).children.lazy.map { $0.value }.generate())
}
```

- We take advantage of Swift's lazy map in order to lazily extract the `value` from each child.
- The return value is wrapped in an `anyGenerator` in order to erase the LazyMapGenerator implementation detail.

We can now iterate over tuples like so:

```language-swift
for val in generatorForTuple(fibonacci) {
  val
}
```

- [View the gist for this function](https://gist.github.com/jverkoey/24ca1fe9cfd76b93d879).
- Learn about how this is used in practice by reading [Enumerate messages of a MIDIPacket in Swift](/enumerate-messages-midipacket-swift-reflection/)
