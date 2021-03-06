# Ember.js Testing Guide

These guidelines are specific to the 1.13.x version of Ember.js

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt)


## Table of Contents

### Test Cases

* [Comments](#comments)
* [Naming: Unit Tests](#naming-unit-tests)
* [Naming: Component Unit Tests](#naming-component-unit-tests)
* [Naming: Component Integration Tests](#naming-component-integration-tests)


### Assertions

* [assert.expect()](#assertexpect)
* [Specificity](#specificity)


### DOM References

* [Element vs jQuery object](#element-vs-jquery-object)


### Components

* [Unit Tests](#component-unit-tests)
* [Integration Tests](#component-integration-tests)


### Rendering Templates

* [Unit and Integration Tests](#rendering-templates-1)


### Examples

* [Unit Tests: Testing default property values](#example-testing-default-property-values)
* [Unit Tests: Testing expected mixins are present](#example-testing-expected-mixins-are-present)
* [Unit Tests: Testing dependent keys are correct](#example-testing-dependent-keys-are-correct)
* [Unit Tests: Testing observer keys are correct](#example-testing-observer-keys-are-correct)
* [Component Unit Tests: Testing there is no DOM-reference leakage outside of a component](https://github.com/softlayer/sl-ember-test-helpers#global-libraries)
* [Component Integration Tests: Testing default rendered state](#example-testing-default-rendered-state)
* [Element vs jQuery object](#example-element-vs-jquery-object)
* [Component Integration Tests: find() and hasClass()](#example-component-integration-tests---find-and-hasclass)
* [Rendering templates](#example-rendering-templates)
* [Asynchronous testing](#example-asynchronous-testing)

---

### Comments

* Test case descriptions **SHOULD** be descriptive enough to void the need for
DocBlocks
* Test cases **MAY** be preceded with JSDoc DocBlocks but it is **RECOMMENDED**
they do not.  Traditional DocBlocks with tags do not add any value, though a
DocBlock containing comments explaining something peculiar about the test case
**MAY** be employed.  Do so sparingly.
* Comments **MAY** be used within test cases but there should rarely be logic
complicated enough that comments are warranted.  These are tests - there should
only be light-weight logic most likely testing input and output or isolated
behaviors.


### Naming: Unit Tests

* Test cases asserting that default properties are assigned to their
expected values **MUST** be named: *Default property values are set correctly*
    * [See Example](#example-testing-default-property-values)
* Test cases asserting that expected Mixins are mixed in **MUST** be
named: *Expected Mixins are present*
    * [See Example](#example-testing-expected-mixins-are-present)
* Test cases asserting that computed properties are observing the
correct properties **MUST** be named: *Dependent keys are correct*
    * [See Example](#example-testing-dependent-keys-are-correct)
* Test cases asserting that observer properties are observing the
correct properties **MUST** be named: *Observer keys are correct*
    * [See Example](#example-testing-observer-keys-are-correct)


### Naming: Component Unit Tests

* Test cases asserting that there is no DOM-reference leakage outside of a
component **MUST** be named: *There are no references to Ember.$, $ or jQuery*
    * [See Example](https://github.com/softlayer/sl-ember-test-helpers#global-libraries)


### Naming: Component Integration Tests

* Test cases asserting the default rendered state of the component **MUST** be
named: *Default rendered state*
    * [See Example](#example-testing-default-rendered-state)

### assert.expect()

* **MUST** only be used in asynchronous tests
* When used it **MUST** be the first line of code in the test case

[See example](#example-asynchronous-testing)


### Specificity

* Assertions **MUST** be the most specific possible
    * in the case of `ember-cli-qunit` this means that `assert.strictEqual()`
    is preferred over `assert.equal()`, for example


### Element vs jQuery object

* If a `const` or `let` assignment directly references a DOM element then the
name used **MUST** end with the string "*Element*"
* If a `const` or `let` assignment references a jQuery reference to a DOM element
then the name used **MUST** not end with the string "*Element*"

[See Example](#example-element-vs-jquery-object)


### Component Unit Tests

* **MUST** test:
    * default values of all properties, including `Enums`, except for: [See Example](#example-testing-default-property-values)
        * `attributeBindings`
        * `classNames`
        * `classNameBindings`
    * that any expected mixins are mixed in
    [See Example](#example-testing-expected-mixins-are-present)
    * the dependent keys computed properties are observing, via the
    `_dependentKeys` property [See Example](#example-testing-dependent-keys-are-correct)
    * the observer keys ember observers are observing, via the
    `__ember_observes__` property [See Example](#example-testing-observer-keys-are-correct)
    * the logic of the computed property functions
    * the logic of any actions
    * the logic of any event handlers
    * that there is no DOM-reference leakage outside of a component
        * This is done in Unit Tests due to the limitation that we cannot invoke specific events or directly call specific functions/methods on the component from within Integration tests
* Whenever testing default values of properties in components, mixins, routes,
etc, you **SHOULD** assert the properties in the same alphabetical order as
they appear in the unit under test.  This makes it easier to find the related
assertion, confirm one hasn't been missed, etc.

Of course there are always exceptions...

* if an action or method only accesses or affects the DOM then it does not need
to be Unit tested and Integration test only coverage is preferred
* an action or method, in addition to accessing or affecting the DOM, which
also interacts with properties on the component, may need to also be Unit tested
    * if the properties being set only pertain to the rendering of DOM, and not
    used by another other logic flow of the component, then Integration test
    only coverage is preferred
    * if the properties being set pertain to the rendering of the DOM and are
    also used by other logic flow the action or method should also be Unit
    tested
        * it is recommended that the logic of the action or method be broken
        into separate method (calls) to encompass more discreet units of work.
        Doing so will then make it very easy to either Unit or Integration test
        an action or method without having to exercise it via both, having to
        contrive data paths to get to the parts desired to be tested.

### Component Integration Tests

* **MUST** test:
    * default rendered state of the component (only populating required
    properties), including: [See Example](#example-testing-default-rendered-state)
        * default classes applied
        * default properties/attributes (values) applied
        * default characteristics of any mixins used by the component
    * any properties that can be provided as different values that cause a DOM
    change
    * any bound values
        * do not need to exercise the entire range of possible logic - this is
        done in the unit test.  Instead, this testing ensures that the template
        has bound things together correctly
    * actions
        * do not need to test the logic of the action - this is done in the
        unit test.  Instead, this testing ensures that the template has bound
        things together correctly
* **MUST** reference a component under test via `this.$( '>:first-child' )`
rather than via a class name when appropriate, which is the majority of the
time.
    * Not using a classname removes the too-tight coupling between the styling
    used and the functionality provided by the component.
    * When this is not always appropriate is when you are rendering multiple
    components and need to access ones beyond `first-child`.
* **MUST** use `find()` for all internal selectors [See Example](#example-component-integration-tests---find-and-hasclass)
* **MUST** use `hasClass()` to assert class existence for all DOM elements that
are not nested inside of the component *AND* are only differentiated by their
class [See Example](#example-component-integration-tests---find-and-hasclass)
* **SHOULD** use `find().length` to `assert.ok()` the existence of a class when
the above is not possible [See Example](#example-component-integration-tests---find-and-hasclass)



### Rendering Templates

* Template strings **MUST** be tagged/prefixed with `hbs` instead of compiled
with `Ember.HTMLBars.compile`
* If a template needs to be registered on the registry it **MUST** also be
unregistered at the conclusion of its use

[See Example](#example-rendering-templates)


### Example: Testing default property values

```javascript
// sl-ember-components/components/sl-alert

import Ember from 'ember';
import layout from '../templates/components/sl-alert';

/**
 * Bootstrap theme names for alert components
 *
 * @memberof module:addon/components/sl-alert
 * @enum {String}
 * @property {String} DANGER 'danger'
 * @property {String} INFO 'info'
 * @property {String} SUCCESS 'success'
 * @property {String} WARNING 'warning'
 */
export const Theme = Object.freeze({
    DANGER: 'danger',
    INFO: 'info',
    SUCCESS: 'success',
    WARNING: 'warning'
});

/**
 * @module
 * @augments ember/Component
 */
export default Ember.Component.extend({

    // -------------------------------------------------------------------------
    // Dependencies

    // -------------------------------------------------------------------------
    // Attributes

    /** @type {String} */
    ariaRole: 'alert',

    /** @type {String[]} */
    classNameBindings: [
        'themeClassName',
        'dismissable:alert-dismissable'
    ],

    /** @type {String[]} */
    classNames: [
        'alert',
        'sl-alert'
    ],

    /** @type {Object} */
    layout,

    // -------------------------------------------------------------------------
    // Actions

    /**
     * @type {Object}
     */
    actions: {

        /**
         * Trigger a bound "dismiss" action when the alert is dismissed
         *
         * @function actions:dismiss
         * @returns {undefined}
         */
        dismiss() {
            this.sendAction( 'dismiss' );
        }

    },

    // -------------------------------------------------------------------------
    // Events

    // -------------------------------------------------------------------------
    // Properties

    /**
     * Whether to make the alert dismissable or not
     *
     * @type {Boolean}
     */
    dismissable: false,

    /**
     * The Bootstrap "theme" style to apply to the alert
     *
     * @type {Theme}
     */
    theme: Theme.INFO,

    // -------------------------------------------------------------------------
    // Observers

    // -------------------------------------------------------------------------
    // Methods

    /**
     * The generated Bootstrap "theme" style class for the alert
     *
     * @function
     * @returns {String} Defaults to "alert-info"
     */
    themeClassName: Ember.computed(
        'theme',
        function() {
            const theme = this.get( 'theme' );

            return `alert-${theme}`;
        }
    )

});
```

```javascript
// component integration test

import { Theme } from 'sl-ember-components/components/sl-alert';

test( 'Default property values are set correctly', function( assert ) {
    const component = this.subject();

    assert.strictEqual(
        component.get( 'ariaRole' ),
        'alert',
        'ariaRole: "alert"'
    );

    assert.strictEqual(
        component.get( 'dismissable' ),
        false,
        'dismissable: false'
    );

    assert.strictEqual(
        component.get( 'theme' ),
        Theme.INFO,
        `theme: "${Theme.INFO}"`
    );
});
```


### Example: Testing expected mixins are present

```javascript
// unit test

import InputBasedMixin from 'your-library/mixins/input-based';

test( 'Expected Mixins are present', function( assert ) {
    assert.ok(
        InputBasedMixin.detect( this.subject() ),
        'The input-based mixin is present'
    );
});
```

### Example: Testing dependent keys are correct

```javascript
// unit test

test( 'Dependent keys are correct', function( assert ) {
    const component = this.subject();

    const currentLabelDependentKeys = [
        'label',
        'pending',
        'pendingLabel'
    ];

    const sizeClassDependentKeys = [
        'size'
    ];

    assert.deepEqual(
        component.currentLabel._dependentKeys,
        currentLabelDependentKeys,
        'Dependent keys are correct for currentLabel()'
    );

    assert.deepEqual(
        component.sizeClass._dependentKeys,
        sizeClassDependentKeys,
        'Dependent keys are correct for sizeClass()'
    );
});
```


### Example: Testing observer keys are correct

```javascript
// unit test

test( 'Observer keys are correct', function( assert ) {
    const component = this.subject();

    const setupTypeaheadKeys = [
        'suggestions'
    ];

    assert.deepEqual(
        component.setupTypeahead.__ember_observes__,
        setupTypeaheadKeys,
        'Observer keys are correct for setupTypeahead()'
    );
});
```


### Example: Testing default rendered-state

```javascript
// sl-ember-components/components/sl-alert

import Ember from 'ember';
import layout from '../templates/components/sl-alert';

/**
 * Bootstrap theme names for alert components
 *
 * @memberof module:addon/components/sl-alert
 * @enum {String}
 * @property {String} DANGER 'danger'
 * @property {String} INFO 'info'
 * @property {String} SUCCESS 'success'
 * @property {String} WARNING 'warning'
 */
export const Theme = Object.freeze({
    DANGER: 'danger',
    INFO: 'info',
    SUCCESS: 'success',
    WARNING: 'warning'
});

/**
 * @module
 * @augments ember/Component
 */
export default Ember.Component.extend({

    // -------------------------------------------------------------------------
    // Dependencies

    // -------------------------------------------------------------------------
    // Attributes

    /** @type {String} */
    ariaRole: 'alert',

    /** @type {String[]} */
    classNameBindings: [
        'themeClassName',
        'dismissable:alert-dismissable'
    ],

    /** @type {String[]} */
    classNames: [
        'alert',
        'sl-alert'
    ],

    /** @type {Object} */
    layout,

    // -------------------------------------------------------------------------
    // Actions

    /**
     * @type {Object}
     */
    actions: {

        /**
         * Trigger a bound "dismiss" action when the alert is dismissed
         *
         * @function actions:dismiss
         * @returns {undefined}
         */
        dismiss() {
            this.sendAction( 'dismiss' );
        }

    },

    // -------------------------------------------------------------------------
    // Events

    // -------------------------------------------------------------------------
    // Properties

    /**
     * Whether to make the alert dismissable or not
     *
     * @type {Boolean}
     */
    dismissable: false,

    /**
     * The Bootstrap "theme" style to apply to the alert
     *
     * @type {Theme}
     */
    theme: Theme.INFO,

    // -------------------------------------------------------------------------
    // Observers

    // -------------------------------------------------------------------------
    // Methods

    /**
     * The generated Bootstrap "theme" style class for the alert
     *
     * @function
     * @returns {String} Defaults to "alert-info"
     */
    themeClassName: Ember.computed(
        'theme',
        function() {
            const theme = this.get( 'theme' );

            return `alert-${theme}`;
        }
    )

});
```

```html
// ../templates/components/sl-alert

{{#if dismissable}}
    <button
        {{action "dismiss"}}
        class="close"
        data-dismiss="alert"
        type="button"
    >
        <span aria-hidden="true">&times;</span>
        <span class="sr-only">Close</span>
    </button>
{{/if}}

{{yield}}
```

```javascript
// component integration test

test( 'Default rendered state', function( assert ) {

    this.render( hbs`
        {{#sl-alert}}
            Default info alert
        {{/sl-alert}}
    ` );

    assert.ok(
        this.$( '>:first-child' ).hasClass( 'alert' ),
        'Has class "alert"'
    );

    assert.ok(
        this.$( '>:first-child' ).hasClass( 'alert-info' ),
        'Default theme class is applied'
    );

    assert.strictEqual(
        this.$( '>:first-child' ).attr( 'role' ),
        'alert',
        'ARIA role is applied'
    );
});
```


### Example: Element vs jQuery object

```javascript
// unit or integration test

test( 'example', function( assert ) {
    const input = this.$( '>:first-child' ).find( 'input' );
    const inputElement = input.get( 0 );
});
```


### Example: Component integration tests - find() and hasClass()

```javascript
// component

export default Ember.Component.extend({

    // -------------------------------------------------------------------------
    // Attributes

    /** @type {String[]} */
     classNames: [
        'componentClass'
    ]
});
```

```html
// template

<p class="help-block">{{helpText}}</p>

<span>Some content</span>

<span class="moreText">Some more</span>
```

```javascript
// component integration test

assert.ok(
    this.$( '>:first-child' ).hasClass( 'componentClass' ),
    'Has expected class'
);

assert.ok(
    this.$( '>:first-child' ).find( '> p' ).hasClass( '.help-block' )
    'Has expected class'
);

assert.ok(
    this.$( '>:first-child' ).find( 'span.moreText' ).length
    'Has more text'
);
```


### Example: Rendering templates

```javascript
// unit or integration test

import hbs from 'htmlbars-inline-precompile';

test( 'Testing something needing template', function( assert ) {
    const template = hbs`
        {{#some-component label="One" name="one"}}
            One
        {{/some-component}}
    `;

    this.registry.register( 'template:test-template',  template );

    ...

    this.registry.unregister( 'template:test-template' );
});
```

### Example: Asynchronous testing

```javascript
// unit or integration test

test( 'Testing asynchronous behavior', function( assert ) {
    assert.expect( 2 );

    const done = assert.async();

    this.$( '.someSelector' ).queue( () => {
        assert.strictEqual(
            ...
        );

        assert.ok(
            ...
        );

        done();
    });
});
```
