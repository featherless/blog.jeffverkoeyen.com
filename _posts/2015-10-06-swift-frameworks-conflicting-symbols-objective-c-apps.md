---
layout: post
---

# Swift frameworks with conflicting symbols in Objective-C apps

What happens when an Objective-C app includes two or more Swift frameworks that happen to have a conflicting class name? **Frustration**.

Swift rightfully doesn't encourage the use of namespace prefixes, meaning this frustration will become increasingly common as more Swift frameworks are open-sourced and used in Objective-C apps.

Let's see a concrete example of the problem and how we can minimize its likelihood.

## The problem

Consider the following pair of frameworks:

```language-swift
// Within Framework1.framework
@objc public class SomeManager: NSObject {
  public func doThing() {
    NSLog(@"SomeManager - Swift - Framework1");
  }
}

// Within Framework2.framework
@objc public class SomeManager: NSObject {
  public func doThing() {
    NSLog(@"SomeManager - Swift - Framework2");
  }
}
```

We might find the following code in an app that includes both frameworks:

```language-objectivec
SomeManager *instance = [SomeManager new];
[instance doThing];
```

Can you guess which class is instantiated? Not likely, because the answer is *undefined*. **The linker will map an arbitrary implementation to the class each time the app is rebuilt**.

## Namespace prefixing

Barring an Objective-C language feature that allows for the distinction of symbols by framework, we need to do the next-best thing: *namespace prefixing*.

Prefixing Objective-C classes is such a common practice that it's automatic for most Objective-C developers. Software designers new to programming and learning Swift may not be familiar with this however, so in short:

> Namespace prefixing is the practice of adding a three or four letter prefix ([Apple has reserved two-letter prefixes](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html)) to the beginning of a symbol exposed by your framework. For example: if your company's name was Lumi you might use LUMI as your prefix. `LUMISomeClass`, `LUMISomeFunc`, or `LUMISomeStruct` would all be examples of symbol names in your framework.

Swift rightfully doesn't encourage the use of namespace prefixes because frameworks **are** the namespace. But Objective-C will treat Swift frameworks like any old Objective-C framework and so that namespacing is lost. Thankfully, Swift provides `@objc(name)`.

`@objc(name)` allows you to provide an Objective-C-specific name for a class to be used only from Objective-C code. If we modify our original example:

```language-swift
// Within Framework1.framework
@objc(FW1SomeManager) public class SomeManager: NSObject {
  public func doThing() {
    NSLog(@"SomeManager - Swift - Framework1");
  }
}

// Within Framework2.framework
@objc(FW2SomeManager) public class SomeManager: NSObject {
  public func doThing() {
    NSLog(@"SomeManager - Swift - Framework2");
  }
}
```

These classes would be referred to in Objective-C as `FW1SomeManager` and `FW2SomeManager` while Swift code could still use `SomeManager` and `Framework1.SomeManager` as before - win win!

## Concluding notes

When writing Swift frameworks that may used by Objective-C apps, **prefix your Swift class and struct names using `@objc()`**. Not doing so in a shared or open source framework can lead to undefined behavior in the event of naming conflicts.

Note that `@objc()` not presently allow you to provide an Objective-C name for `enum` types.

---

### llvm feature request

In the absence of prefixed Swift symbols, a linker warning would be a helpful tool for identifying situations where the linker had to choose between duplicate symbols. Sadly, **I could not find any such linker warning**. This would be a welcome addition to llvm so I've [filed a feature request](https://llvm.org/bugs/show_bug.cgi?id=25083).

---

In exploring solutions to this problem I found out that you *can* refer to a specific Swift framework class using NSClassFromString.

> Disclaimer: I do not encourage the use of this fact in a production application.

For example, `NSClassFromString(@"Framework2.SomeManager")` allows you to instantiate Framework2's SomeManager. This approach can't be checked at compile-time and may be subject to changes in future Swift releases, so its **utility is limited**. This does **not** work for Objective-C classes contained within frameworks.

### References

<blockquote class="twitter-tweet" data-conversation="none" lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/featherless">@featherless</a> You can namespace classes in Objective-C only with @ objc(ABCMyClass)</p>&mdash; Scott Berrevoets (@ScottBerrevoets) <a href="https://twitter.com/ScottBerrevoets/status/651460908363857920">October 6, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
