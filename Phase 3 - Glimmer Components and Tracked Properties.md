# Phase 3 – Glimmer components and `@tracked` properties

## Overview

There are three new features that comprise the final set of features in Ember Octane:

- Glimmer components
- Autotracking

In combination, these can dramatically improve developer experience *and* performance!

More than any earlier phases, Glimmer components and autotracking require learning a *new mental model*: expect an adjustment period as you unlearn old habits and learn new (simpler, nicer!) ones in their place.

### Glimmer components

Glimmer components are a simpler, lighter-weight, faster, easier-to-use component class which apply the lessons learned from many years of working with Ember components. Key differences:

- Most lifecycle hooks are removed: the class has only `constructor` and `willDestroy`.
- Arguments to the component are namespaced:
    - in templates, as `@foo` instead of `foo` or `this.foo`
    - in classes, as `this.args.foo` instead of just `this.foo`
- There is no two-way binding, so DDAU is strictly enforced.

### Autotracking

Autotracking is a next-generation *reactivity system* for Ember, with several key benefits:

- **Lower annotation burden for developers.** Mark root state with `@tracked` once, and you never have to specify computed property dependent keys—you can just use native getters!
- No need for `get` or `set` for autotracked state. Normal property access (`const wat = this.foo + 42;`) and assignment (`this.bar = 123;`) just work!
- **Clear ownership of state in the program.** Switching to autotracking prevents many bugs which plagued Ember Classic’s observer-based, shared-ownership model.

Autotracking enforces one-way data flow, making it a perfect complement to Glimmer components. As a bonus, it works well with both object-oriented and functional styles!

## Migration

Migrating to Glimmer components and autotracking is straightforward once you have completed Octane Phases 1 and 2. However, you should keep the following in mind:

1. **Convert to Glimmer components and autotracking at the same time.** Changing the base class and converting to using `@tracked` instead of decorators both require fully understanding the component and its data flow. Do that just *once*!

    * Doing Glimmer Components and autotracking “at the same time” does not mean this has to be a single RB. It might! But it also might not! Refactor safely, and take intelligent risks here. It *does* mean that you should migrate each component to both the Glimmer Component base class and autotracking *before moving on*.

    * This process will vary enormously. With some components, you’ll find that you just remove a bunch of `@computed` decorators, change a few things to reference `this.args.foo` instead of `this.foo` and you’re done. With others, you’ll end up rewriting massive chunks of the functionality as a much smaller custom modifier instead. Sometimes you’ll get rid of the backing class entirely. Sometimes you’ll end up finding that it makes sense to break things into multiple components. **The key is that you have to use your judgment as a team of engineers—_there is no one-size-fits-all solution_.**

    * To make this easier, there is a [codemod](https://github.com/ember-codemods/ember-tracked-properties-codemod) to convert some simple cases to tracked properties. Take advantage of it!

2. **Start with “leaf” components.** computed properties can’t depend on native getters without extra annotations (which we’re trying to get away from), so start by converting to autotracking in components which don’t pass their properties to *other* components.

3. **You may still occasionally need `set` and `get` to interact with third-party code.** If you are using an addon which has not been updated to use autotracking, you need to use `get` to retrieve any values which were in the dependency keys from that addon before.

## References

Ember guides:

- [Octane announcement blog post](https://blog.emberjs.com/2019/12/20/octane-is-here.html) – see especially the discussions of Glimmer components and `@tracked`
- [Components](https://guides.emberjs.com/release/components/)
- In-Depth Topics:
    - [Patterns for Components](https://guides.emberjs.com/release/in-depth-topics/patterns-for-components/)
    - [Autotracking In-Depth](https://guides.emberjs.com/release/in-depth-topics/autotracking-in-depth/)

Other resources:

- [Coming Soon in Ember Octane – Part 5: Glimmer Components (Chris Garrett)](https://www.pzuraq.com/coming-soon-in-ember-octane-part-5-glimmer-components/)
- Ember Atlas:
    - [Glimmer Components](https://www.notion.so/Glimmer-Components-42adc94830ed4ed6be1af13bb6b7eacb)
    - [Tracked properties](https://www.notion.so/Tracked-Properties-3c5b4e0943a34c52894bb5a39e41c263)
- Chris Garrett’s series on autotracking:
    - [What is Reactivity?](https://www.pzuraq.com/what-is-reactivity/)
    - [What Makes a Good Reactive System](https://www.pzuraq.com/what-makes-a-good-reactive-system/)
    - [How Autotracking Works](https://www.pzuraq.com/how-autotracking-works/)
    - [Autotracking Case Study](https://www.pzuraq.com/autotracking-case-study-trackedmap/)
- Chris Krycho’s posts on Octane and autotracking:
    - [Migrating Off of `PromiseProxyMixin` in Ember Octane](https://v5.chriskrycho.com/journal/migrating-off-of-promiseproxymixin-in-ember-octane/)
    - [Async Data and Autotracking in Ember Octane](https://v5.chriskrycho.com/journal/async-data-and-autotracking-in-ember-octane/)
    - [Autotracking: Elegant DX Via Cutting-Edge CS](https://v5.chriskrycho.com/journal/autotracking-elegant-dx-via-cutting-edge-cs/)
    - [Understanding `args` in Glimmer Components](https://v5.chriskrycho.com/journal/understanding-args-in-glimmer-components/)
    - [Patterns for “Smart” Components in Ember](https://v5.chriskrycho.com/journal/patterns-for-smart-components-in-ember/)
