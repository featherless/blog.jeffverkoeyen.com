---
layout: post
---

# Get current Xcode playground page name

You can get the name of the current Playground page using the following snippet:

```language-swift
NSProcessInfo.processInfo().environment["PLAYGROUND_NAME"]
```
