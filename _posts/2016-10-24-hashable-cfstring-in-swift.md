---
layout: post
---

# Hashable CFString in Swift

> This post is also available as a Swift playground at https://github.com/jverkoey/playgrounds.

If you find that you need to use a CFString in a Swift switch statement you'll likely run into the following cryptic error:

    Expression pattern of type 'CFString' cannot match values of type 'CFString'

or the slightly more helpful error when attempting to use a Set<CFString>:

    Type 'CFString' does not conform to protocol 'Hashable'

So let's make CFString [Hashable].

## CFString+Hashable.swift

```language-swift
extension CFString : Hashable {
  public var hashValue: Int { return Int(CFHash(self)) }
}

public func ==(lhs: CFString, rhs: CFString) -> Bool {
  return CFStringCompare(lhs, rhs, CFStringCompareFlags()) == .CompareEqualTo
}
```

- We take advantage of [CFHash] to implement the hashValue.
- The `==` operator is a simple mapping to [CFStringCompare].

[View the complete code on GitHub](https://gist.github.com/jverkoey/afc73edaf0f60fc180ad)

[Hashable]: http://swiftdoc.org/v2.0/protocol/Hashable/
[CFHash]: https://developer.apple.com/library/prerelease/ios/documentation/CoreFoundation/Reference/CFTypeRef/index.html#//apple_ref/c/func/CFHash
[CFStringCompare]: https://developer.apple.com/library/mac/documentation/CoreFoundation/Reference/CFStringRef/#//apple_ref/c/func/CFStringCompare
