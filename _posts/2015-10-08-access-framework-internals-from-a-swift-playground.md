---
layout: post
---

# How to access framework internals from a Swift Playground

[Swift 2 introduced](https://developer.apple.com/library/prerelease/ios/documentation/DeveloperTools/Conceptual/WhatsNewXcode/Articles/xcode_7_0.html) the `@testable` keyword so that unit test targets could access internal APIs of frameworks like so:

```language-swift
@testable import MyFramework
```

Without this keyword the only alternative was to make APIs public, defeating the purpose of [access controls].

So check this out: **`@testable` also works in Playgrounds**!

Paired with elegant use of the inline documentation renderer, it's now possible to write Playgrounds that function as **interactive internal documentation**. This sets the stage for great new forms of technical documentation.

[access controls]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html
