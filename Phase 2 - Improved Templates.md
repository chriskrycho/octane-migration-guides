# Phase 2 – Improved Templates

## Overview

Octane makes templates clearer, more maintainable, and easier to codemod!

In classic Ember, everything except plain HTML looked pretty much the same:

- component invocation: `{{profile}}`
- values passed as arguments to the component: `{{user}}`
- values from the component backing class: `{{fullName}}`
- values from block yields: `{{yieldedValue}}`
- helper invocation: `{{add base value}}`

Octane clarifies our templates by making all of these unambiguous and distinct.

- component invocation: [`<UserProfile @user={{@model}} />`][abc] ([angle-bracket component invocation syntax][abc])
- values passed as arguments to the component: [`{{@user}}`][named-args] ([named arguments][named-args])
- values from the component backing class: [`{{this.fullName}}`][strict-this] ([explicit `this` in templates][strict-this])
- values from block yields: `{{yieldedValue}}` (unchanged from classic Ember, but now unambiguous because of the other changes)
- helper invocation `{{add this.base @value}}` (unchanged from classic Ember, but now unambiguous because of the other changes)

Additionally, Octane makes two other changes to improve the clarity and maintainability of our templates:

- [`outerHTML` semantics for templates](#outerhmtl-tagless-components)
- a new primitive for interacting with templates: [element and render modifiers](#modifiers).

[named-args]: #named-arguments
[strict-this]: #explicit-this-in-templates
[abc]: #angle-bracket-invocation

### Example

### Invocation

Before:

```handlebars
{{#each todoItems as |todo|}}
  {{todos$todo/checkbox
    completed=todo.completed
    onToggleCompleted=(action toggleCompleted todo)
    data-test-todo-checkbox=todo.id
  }}
  <p>{{todo.description}}</p>
{{/each}}
```

After:

```handlebars
{{#each this.todoItems as |todo|}}
  <todos$Todo::Checkbox
    @completed={{todo.completed}}
    @onToggleCompleted={{fn this.toggleCompleted todo}}
    data-test-todo-checkbox={{todo.id}}
  />
  <p>{{todo.description}}</p>
{{/each}}
```

### Implementation

Before:

```javascript
import Component from '@ember/component';
import { action, toggleProperty } from '@ember/object';
import { tagName, classNames, attributeBindings } from '@ember-decorators/component';
import classic from 'ember-classic-decorator';

@classic
@tagName('label')
@classNames('todo-item__checkbox')
@attributeBindings('data-test-todo-checkbox')
export default class TodoCheckbox extends Component {
  @computed('completed')
  get label() {
    return this.completed ? 'completed' : 'todo';
  }
}
```

```handlebars
{{label}}
<input
  type='checkbox'
  checked={{this.completed}}
  {{action toggleCompleted on="click"}}
/>
```

After:

```javascript
import Component from '@ember/component';
import { action, toggleProperty } from '@ember/object';
import { tagName } from '@ember-decorators/component';
import classic from 'ember-classic-decorator';

@classic
@tagName('')
export default class TodoCheckbox extends Component {
  @computed('completed')
  get label() {
    return this.completed ? 'completed' : 'todo';
  }
}
```

```handlebars
<label class='todo-item__checkbox' ...attributes>
  {{this.label}}
  <input
    type='checkbox'
    checked={{this.completed}}
    {{on "click" @onToggleCompleted}}
  />
</label>
```

Or, go a step further and remove the backing class entirely (which has substantially better performance!):

```handlebars
<label class='todo-item__checkbox' ...attributes>
  {{if @completed 'completed' 'todo'}}
  <input
    type='checkbox'
    checked={{this.completed}}
    {{on "click" @onToggleCompleted}}
  />
</label>
```

**NOTE**: If your backing class is empty and looks something like the example below, you should convert its template to use Octane syntax and then **delete** the backing JS.

```js
import Component from '@ember/component';
export default class MyComponent extends Component {};
```

## Angle bracket invocation

This syntax brings component invocation in line with normal HTML syntax, and helps distinguish between helpers, components and properties. It also paves the path for Glimmer components’ distinction between component properties and arguments.

### Conversion

To convert your templates to angle bracket syntax, run the [ember-angle-brackets-codemod](https://github.com/ember-codemods/ember-angle-brackets-codemod), by running `ember serve` and then running the codemod against the running application.

As a one-off:

```sh
$ npx ember-angle-brackets-codemod http://localhost:4200/tests/index.html?filter=ZZZZZ <path>
```

Using [Volta] to install a default version of the codemod so you can run it repeatedly without reinstalling its dependencies:

[Volta]: https://volta.sh

```sh
$ volta install ember-angle-brackets-codemod
$ ember-angle-brackets-codemod http://localhost:4200/tests/index.html?filter=ZZZZZ <path>
```

Here `<path>` is the path on disk to the files you want to convert: a folder, an individual file, etc.

### References

- [Ember Guides: Components](https://guides.emberjs.com/release/components/)
- [Ember Octane vs. Classic Cheatsheet: Angle Brackets Component Invocation](https://ember-learn.github.io/ember-octane-vs-classic-cheat-sheet/#component-templates)
- Ember Atlas for [Angle Bracket migration](https://www.notion.so/Angle-Bracket-Syntax-c84839ea6caa4dd1bb4640d23a8890f2)
- [Ember Angle Brackets Codemod](https://github.com/ember-codemods/ember-angle-brackets-codemod)

## Named arguments

- Arguments passed to components in angle-bracket invocation are prefixed with `@`
- Arguments used within a component template are *also* be prefixed with `@`. (It works either way for Ember Components, but `@` is *required* for Glimmer components, so should be used in both for consistency.)

There is no codemod for this, because it is impossible to tell statically whether a given item is an argument to a component or not.

### References

- [Ember Guides: Components](https://guides.emberjs.com/release/components/)
- [Ember Octane vs. Classic Cheatsheet: Angle Brackets Component Invocation](https://ember-learn.github.io/ember-octane-vs-classic-cheat-sheet/#component-templates)
- [Ember Atlas: Ember Octane: Angle Bracket Syntax](https://www.notion.so/Angle-Bracket-Syntax-c84839ea6caa4dd1bb4640d23a8890f2)
- [Coming Soon in Ember Octane - Part 2: Angle Bracket Syntax & Named Arguments](https://blog.emberjs.com/2019/02/19/coming-soon-in-ember-octane-part-2.html)

## Explicit `this` in templates

In Octane, all references to properties on the backing class must be prefixed with `this`, like `{{this.greeting}}`.

### Conversion

For a one-off run:

```sh
$ npx ember-no-implicit-this-codemod https://pemberly.www.linkedin.com:4443/tests/index.html?filter=XXXX <path> prefix-component-properties-only=true
```

Using [Volta] to install a default version of the codemod so you can run it repeatedly without reinstalling its dependencies:

```sh
$ volta install ember-no-implicit-this-codemod
$ ember-no-implicit-this-codemod https://pemberly.www.linkedin.com:4443/tests/index.html?filter=XXXX <path> prefix-component-properties-only=true
```

Running the test with `?filter=XXXX` lets us load the entire app via the test suite, without wasting time for  Here `<path>` is the path on disk to the files you want to convert: a folder, an individual file, etc.

_**Note:** the `prefix-component-properties-only` flag ensures correctness of prefixing only local properties of the template's backing JS file. Refer to this [document](https://docs.google.com/document/d/1EbtA3uJVxwxxxku4ImANGBmUeLFD6wA3Yz__LPng6i8/edit?usp=sharing) for more information, and prefer it as soon as it becomes available._

### References

- Ember Atlas - [Required `this` in templates](https://www.notion.so/Required-this-in-Templates-917fbec133154873b18bb92cb5af2e62)
- [No Implicit This codemod](https://github.com/ember-codemods/ember-no-implicit-this-codemod)
- [Coming Soon in Ember Octane - Part 2: Angle Bracket Syntax & Named Arguments](https://blog.emberjs.com/2019/02/19/coming-soon-in-ember-octane-part-2.html)
- [RFC #0496: Handlebars Strict Mode](https://github.com/emberjs/rfcs/blob/master/text/0496-handlebars-strict-mode.md)

### Why is not every property prefixed with `this` in my template?

With `prefix-component-properties-only` flag turned on (which is the preferred setting when enabled in the codemod), the codemod will only prefix the owned properties of the template's backing JS class - this ensures the *correctness* of the prefixing and allows you to supply `@` for arguments instead. For less stricter version, remove the flag and it will prefix `this` to *almost* all the properties of the template.

## `outerHTML`/“tagless” components

Convert all component templates from classic Ember’s `innerHTML` semantics—where the template represents the interior HTML and the backing class specifies the wrapping element, classes, attributes, etc.—to `outerHTML` semantics, where the template defines *everything* about the rendered HTML.

The split between JavaScript and Handlebars for defining the template of a component is confusing, and Glimmer components put all HTML information in the template. Combined with [angle bracket syntax](#angle-bracket-invocation), this makes templates clearer *today* and makes them Glimmer-ready.

HTML attributes like `id`, `class`, and `data-test-*` are *forwarded* to the target using `...attributes`.

### Conversion

For a one-off run:

```sh
$ npx tagless-ember-components-codemod <path to directory or component>
```

Using [Volta] to install a default version of the codemod so you can run it repeatedly without reinstalling its dependencies:

```sh
$ volta install tagless-ember-components-codemod
$ tagless-ember-components-codemod <path to directory or component>
```

Note that the codemod intentionally does not convert components where either of the following are true:

- its backing class is a subclass of some other class besides `Component`—including other components
- it uses any of the built-in event-handling methods like `click()`, `mouseDown()`, etc.

These are not *safe* for it to convert automatically:

- The codemod would need to know all the details of all parent classes (including any mixins in the chain) to be able to safely transform the tag, attributes, etc.
- Built-in event handling methods use Ember's events, rather than native DOM events. (For a good overview of the relationship between the two, see [this EmberConf 2018 talk](https://www.youtube.com/watch?v=G9hXjjHFJVs).)

The recommendation for dealing with these is:

- replace component sub-classing with composition of components: extract the common behavior into components which can *wrap* or *yield* for other components
- manually migrate each use of `click()`, `mouseDown()`, etc. to using the `{{on}}` [modifier](#modifiers)

### References

- [Ember Octane vs. Classic Cheatsheet: All Components are Tagless](https://ember-learn.github.io/ember-octane-vs-classic-cheat-sheet/#section-tagless-components)
- [Ember Guides: Components](https://guides.emberjs.com/release/components/)
- [ember-tagless-components-codemod](https://github.com/ember-codemods/tagless-ember-components-codemod)

## Modifiers

Modifiers bridge between Ember Octane’s one-way data flow paradigm and the imperative APIs for the DOM, including binding events, setting up third-party libraries, and more. They are one common replacement for mixins which provided for shared functionality targeting elements in templates.

The most common modifier you will see is `{{on}}`, which takes an event name and a function as its arguments, and replaces the `{{action}}` element modifier. You will often use it in conjunction with the `{{fn}}` helper, which replaces the `{{action}}` helper’s support for partial application of arguments. (During conversion to angle bracket syntax, you will also see many uses of `{{did-insert}}`, `{{did-update}}`, and `{{will-destroy}}`, which replace the `didInsertElement`, `didReceiveAttrs`, and `willDestroy` hooks.

### Notes

1. Modifiers do not run in Fastboot/SSR! They will only run in the booted app. This means you do not need `IS_BROWSER` checks in modifier code; it also means you should *not* put critical business logic or change DOM *contents* with modifiers.
2. There is no *specific* codemod for modifiers. Instead, they are sometimes supplied by *other* codemods.
3. Do *not* use the ember-render-modifiers’ `{{did-insert}}`, `{{did-update}}`, or `{{will-destroy}}` modifiers for *new* code. Instead, if and as you identify specific patterns, rework them in other ways—
    - dedicated modifiers that accomplish a specific task, using [ember-modifer](https://github.com/ember-modifier/ember-modifier)
    - using true one-way data flow with computed properties or native getters and actions

### References

- [Ember Guides: Components: Component State and Actions: HTML Modifiers and Actions](https://guides.emberjs.com/release/components/component-state-and-actions/#toc_html-modifiers-and-actions)
- [Ember Guides: Components: Template Lifecycle, DOM, and Modifiers: Event Handlers](https://guides.emberjs.com/release/components/template-lifecycle-dom-and-modifiers/#toc_event-handlers)
- [Coming Soon in Ember Octane - Part 4: Modifiers](https://blog.emberjs.com/2019/03/06/coming-soon-in-ember-octane-part-4.html)
- [Ember Atlas: Ember Octane: `{{action}}` → `{{on}}` and `{{fn}}`](https://www.notion.so/action-on-and-fn-d56169da46934d5790c800ecff6e76c5)
