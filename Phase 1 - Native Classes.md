# Phase 1 – Native classes and Project Mixout

## What does this mean for you?

You get to use native JavaScript classes for everything in Ember now: components, services, routes, controllers, and class-based helpers!

1. Please write all new classes with native class syntax: `export default class MyComponent extends Component { ... }` instead of `export default Component.extend({ ... })`. For details, see:
    - the updated Ember guides for [routes](https://guides.emberjs.com/release/routing/), [controllers](https://guides.emberjs.com/release/routing/controllers/), and [services](https://guides.emberjs.com/release/services/)
    - [the Native Classes page on the Ember Atlas](https://www.notion.so/Native-Classes-55bd67b580ca49f999660caf98aa1897) (note that the Ember Atlas is not an authoritative source in general, but these particular pages are well-vetted and useful)
    - [Chris Garrett’s blog post on native classes](https://www.pzuraq.com/coming-soon-in-ember-octane-part-1-native-classes/)

2. Convert existing code using the [ember-native-classes-codemod](https://github.com/ember-codemods/ember-native-class-codemod). For more details, see [the codemod README](https://github.com/ember-codemods/ember-native-class-codemod/blob/master/README.md).

3. Actively eliminate mixins from your codebase. While they are still supported by the framework, they are not a part of its future and are likely to be deprecated soon, they do not work with Glimmer components, and everything they provide can be accomplished other ways.

    See [this Ember forum post by Chris Krycho](https://discuss.emberjs.com/t/best-way-to-replace-mixins/17395/2?u=chriskrycho) and [this follow-on post on the same thread](https://discuss.emberjs.com/t/best-way-to-replace-mixins/17395/14?u=chriskrycho) for strategies for migration.
