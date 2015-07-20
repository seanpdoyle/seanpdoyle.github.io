---
layout: post
title: Destructuring Parameters in Ember.Helpers
modified: 2015-07-20
---

With the recent release of [Ember.js version 1.13][ember-1.13]
comes support for a new API for declaring helpers.

One approach creates a helper class via [Ember.Helper]'s `.extend`,
while the other declares the helper with a shorthand API
via the [Ember.Helper.helper] function.

[ember-1.13]: http://emberjs.com/blog/2015/06/12/ember-1-13-0-released.html
[Ember.Helper]: http://emberjs.com/api/classes/Ember.Helper.html
[Ember.Helper.helper]: http://emberjs.com/api/classes/Ember.Helper.html#method_helper
[new-helper]: http://emberjs.com/blog/2015/06/12/ember-1-13-0-released.html#toc_new-ember-js-helper-api

At the core of both approaches is the `compute` function.

This function accepts an array of parameters, as well as a hash of options,
populated when invoked in a Handlebars template:

```handlebars
{% raw %}
{{address "1600 Pennsylvania Ave" "Washington, DC" "20500" separator="<br>"}}
{% endraw %}
```

The corresponding JavaScript would look like this:

```js
import Ember from 'ember';

export default Ember.Helper.helper(
  function compute(params, hash) {
    const street = params[0];
    const cityAndState = params[1];
    const zipCode = params[2];
    const separator = hash.separator;

    // ... do the work
  }
);
```

Using [ES2015 destructuring][es2015], the variable declarations can be moved to
the `function`'s parameter list:

```js
import Ember from 'ember';

export default Ember.Helper.helper(
  function compute([street, cityAndState, zipCode], { separator }) {
  // ... do the work
});
```

While this works great for `Ember.Helper`, there is nothing stopping you from
using this anywhere your functions accept arrays of arguments or hashes of
options:

```js
function calculateArea({ width, height }) {
  return width * height;
}

// call it with a hash
const rectangle = { width: 5, height: 4 };

calculateArea(rectangle); // => 20
```

[es2015]: https://babeljs.io/docs/learn-es2015/#destructuring

Further Reading
---------------

* [Mozilla's Destructuring Assignment documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
* [Replace CoffeeScript with ES6](https://robots.thoughtbot.com/replace-coffeescript-with-es6)
