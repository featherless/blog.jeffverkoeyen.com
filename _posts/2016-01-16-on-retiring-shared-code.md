---
layout: post
---

# Shared code series: retirement

*This is part of [a series on Shared code](/shared-code-series/)*.

**Creating** shared code is easy. **Retiring** shared code, however, is not.

**Creating** shared code is easy because doing so does not *require*
consideration. You are not required to justify the creation of a new API or the addition of a new dependency.

**Retiring** shared code, however, *does* require consideration. To remove an API without going through a deprecation/release process is to cause havoc on clients.

> I use **retire** to mean the act of removing something from shared code. The removed thing might be an API, a dependency, a behavior, or the shared code itself.

In practice, a significant portion of the lifetime of shared code is spent on its retirement. When building shared code for the first time it is remarkably easy not to plan for eventual retirement - you simply haven't had to perform a retirement yet.

For this reason, the purpose of this article is to argue that **planning for retirement** should be an important part of the design of shared code.

---

### Why retirement planning is important for shared code

I've seen two common outcomes when retirement planning is not part of the software design process: **turnover** and **lock-in**.

**Turnover** is when code is "thrown away" in favor of new solutions. This is similar &mdash; but not identical &mdash; to [Not-Invented Here](https://en.wikipedia.org/wiki/Not_invented_here) thinking.

**Lock-in** is when a team does not have the bandwidth to invest in new solutions. This often results in a reduction of engineering velocity as the team attempts to solve new problems within the constraints of older technology.

A conscientious software designer aims to mitigate these outcomes by applying effective **retirement strategies** to their shared code.

### Strategies for retirement

Retirement strategies play a significant role in the long-term maintainability and velocity of shared code. The successful application of the following strategies requires diligence and great care, but the results can be significant.

#### Strategy: Minimize the public API surface area

Why? Assume that every exposed public API will eventually be retired by something newer.

APIs should be expected to fight for their existence.

One approach to this strategy is to put new APIs through an *API review*. Members of your community would be expected to ask questions that justify the exposure of the new APIs. The goal of this vetting process is to find the minimal API required to solve the set of known problems.

#### Strategy: Aim for dependency-less components

Why? Assume that any dependency will eventually be retired.

Each dependency locks a piece of code into a particular set of assumptions. Dependency-less code bears the least risk of eventual retirement. If dependencies are necessary, create meta components that bind together the dependencies; even if the dependencies are retired, your underlying code will be unaffected.

#### Strategy: Draw hard lines between components

Why? It's often far too easy to add cross-component dependencies.

A hard line is a step that introduces friction to referencing a new dependency. This extra friction is intended to force the software designer to justify the new dependency.

Peer review can help here, but it's important that everyone on your team understands the impact of adding new dependencies.

A good example of a hard line is requiring an explicit dependency to be added via a dependency management system. Ideally peer review would catch this and ask to justify the new dependency.

A bad example of a hard line would be code that can be freely imported from any source.

#### Strategy: Avoid "grab bag" components

Why? Grab bag components inevitably grow in a difficult-to-control manner.

This happened in Three20, it happened in NimbusKit, it happens with shared iOS code at Google. The existence of "Core" in all of these cases has lead to a proliferation of APIs in a grab-bag location. Your Core won't be different.

The recommendation is to simply *not* have a "Core" component. Prefer reasonable code duplication over centralization and reap the long-term benefits of dependency-less components.

### Addendum: when is retirement necessary?

Retirement is necessary in any of the following situations:

- A modern alternative solution now exists.
- A better architecture has become apparent.
- The requirements have changed, requiring new APIs and removal of old ones.

Assume that most code you write will need to be retired at some point in the future. The one doing the retiring in the future &mdash; who may not be you! &mdash; will thank you.
