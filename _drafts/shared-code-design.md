# On designing shared software

I will make the case that designing software to be shared leads to higher quality code that's easier to maintain.

# Lifecycle of shared software

Shared software goes through three phases of life:

1. Integration
2. Use
3. Removal

# Levels of Convenience

Convenience is often one of the primary reasons why shared software becomes difficult to maintain or extract.

Every "convenience" API provides some additional level of lock-in to the shared code.

If you're building code on a platform and you intend to replace the platform APIs, your goal should be to not only provide value in its daily use, but to also minimize the cost of integration and removal.

You should not provide convenience APIs. If clients want to build their own convenience APIs then that is up to them. But you as a shared software designer will not invest time into that. Your core APIs should be flexible enough that convenience APIs can easily be built.

- convenience at framework level, app level
- important not to let app convenience leak into framework

Software design is hard, so it's only natural to make every effort to make it as convenient as possible.

Outline:

- Components designed to solve a problem
- Complex setup often turns into "convenience" which ends up making it harder to retire a component
- setup and retirement important to design for
