---
layout: post
---

# Minimal Swift 2.1 protocol conformance

With the advent of protocol-oriented programming Swift types are able to adopt all sorts of super powers with minimal work.

But sometimes it's not clear what that minimal work is. SequenceType, for example, simply requires that you implement `func generate()`; you get everything else for free.

This article provides copy-pastable snippets that represent the minimal implementation required to conform to a given protocol. Please feel free to suggest additional implementations by tweeting [@featherless](http://twitter.com/featherless) with a GitHub gist.

## ArrayLiteralConvertible

> Conforming types can be initialized with array literals.

```language-swift
<# type #> <# extendedType #> : ArrayLiteralConvertible {
  required init(arrayLiteral elements: <# elementType #>...) {
    self.<# storage #> = elements
  }
}
```

Note that this init method can't be defined in an extension.

## CollectionType

> Conformity to `CollectionType` simply requires implementing the methods of `Indexable`.

```language-swift
extension <# extendedType #> : CollectionType {
  var startIndex: <# indexType : ForwardIndexType #> { return <# startIndex #> }
  var endIndex: <# indexType : ForwardIndexType #> { return <# endIndex #> }
  subscript (index: <# indexType : ForwardIndexType #>) -> <# elementType #> {
    return <# subscript value #>
  }
}
```

- `indexType` will usually be some type of Int which is a `ForwardIndexType`.

## Hashable

> Conforming types may be used in `switch` statements and as the element of a `Set`.

```language-swift
extension <# extendedType #> : Hashable {
  var hashValue: Int { return <# hashValue #> }
}

func ==(lhs: <# extendedType #>, rhs: <# extendedType #>) -> Bool {
  return <# comparator #>
}
```

## SequenceType

> Conforming types may be used in `for-in` statements.

```language-swift
extension <# extendedType #> : SequenceType {
  func generate() -> <# generatorType #><<# elementType #>> {
    <# generator state #>

    return anyGenerator {
      if <# terminal condition #> {
        return nil
      }

      <# iteration logic #>

      return nil
    }
  }
}
```

- A collection is a sequence, but a sequence isn't necessarily a collection.

## Protocol missing from this list?

Tweet [@featherless](http://twitter.com/featherless) with a gist to suggest new protocols that can easily be conformed to and it will be added to this list.
