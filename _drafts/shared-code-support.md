---
layout: post
---

What is a support contract? How is it related to public/private APIs?

Anything in Objective-C is technically "public", but it's important for us to make a distinction between what we _intend_ to support and what we don't.

Consider, for example, UIKit. Apple intends to support every public API. They do not intend to support private APIs.

You're free to use Apple's private APIs, of course, but you do so bearing the risk of breakage in the future if Apple decides to change those private APIs. When Apple works on the next release of their OS they will _not_ go out of their way to fix apps that broke because of private API usage. On the other hand, Apple (and any other OS manufacturer) will often go out of their way to ensure that public APIs don't break.

This same concept of "intent to support" applies in this case. Retiring APIs is expensive (go/on-retiring-shared-code), so we are highly incentivized to introduce the minimal possible API that balances getting the job done with ease-of-use.

We use umbrella headers, private/ directories, and other signals to communicate the level of support we intend to give.

In this case, we might have a single header file (which is also the umbrella header): ThirdPartyAttributions.h with a C function prototype.

The implementation details might go in a private/ subdirectory. Someone can certainly import these private headers and use them, but we will not support these use cases.
