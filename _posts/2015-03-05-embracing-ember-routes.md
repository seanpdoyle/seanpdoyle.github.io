---
layout: post
title: "Embracing Ember: Routes"
modified: 2015-03-05
---

[Modal dialogs][modal] are a common UI component
often used to draw attention to
vital pieces of information.

[FormKeep's][formkeep] payment flow involves
presenting a Stripe payment form embedded within a [modal dialog][modal].

![FormKeep's payment
modal](https://images.thoughtbot.com/embracing-ember/formkeep-modal.png)

We went through a few implementations, each of which involved tradeoffs between
user experience and code maintainability.

[formkeep]: https://formkeep.com
[modal]: http://en.wikipedia.org/wiki/Modal_window

Managing visibility with internal state
---------------------------------------

The initial implementation involved the `PayController` managing
an internal `showModal` flag, toggled with a `<a>` tag.

`templates/state.hbs`:

```handlebars
<a href="#" {{action "showModal"}}>Open Stateful Modal</a>

{% raw %}
{{#if showModal}}
  {{#x-modal}}
    {{credit-card card=model}}
  {{/x-modal}}
{{/if}}
{% endraw %}
```

`controllers/state.js`:

```javascript
App.StateController = Ember.Controller.extend({
  showModal: false,

  actions: {
    closeModal: function() {
      this.set('showModal', false);
    },

    showModal: function() {
      this.set('showModal', true);
    },

    saveCard: function(card) {
      /* save to Stripe here */
      this.set('showModal', false);
    }
  }
});
```

This functioned well enough, but had some drawbacks.

From a maintenance perspective:

* Our controller was responsible for maintaining the visibility flag.
* Our controller required an action to mutate the modal's visibility.
* Our template wrapped the modal within a guard conditional.

From a usability perspective:

* Modal visibility state was lost during browser navigations and refreshes.

Thanks to Ember's emphasis on
[serializing application state to the URL][ember-url],
we had options.

[ember-url]: http://tomdale.net/2012/05/ember-routing/

Managing visibility with a query parameter
------------------------------------------

Exposing the modal's visibility flag to the URL was the simplest improvement we
could make.

One possible approach was to embed the flag as a [query
parameter][query-params].

Opening the payment modal would append `?showModal=true` to the browser's URL,
while closing it would remove `?showModal=true`.

[query-params]: http://emberjs.com/guides/routing/query-params/

`templates/query-params.hbs`:

```handlebars
{% raw %}
{{#link-to "query-params" (query-params showModal=true)}}
  Open Query Params Modal
{{/link-to}}

{{#if showModal}}
  {{#x-modal}}
    {{credit-card card=model}}
  {{/x-modal}}
{{/if}}
{% endraw %}
```

`controllers/query-params.js`:

```javascript
App.QueryParamsController = Ember.Controller.extend({
  queryParams: ['showModal'],

  showModal: false,

  actions: {
    closeModal: function() {
      this.set('showModal', false);
    },

    showModal: function() {
      this.set('showModal', true);
    },

    saveCard: function(card) {
      /* save to Stripe here */
      this.set('showModal', false);
    }
  }
});
```

This approach improved upon our previous implementation:

* Users could link to, bookmark, navigate away from, or refresh the page while
  maintaining consistent modal visibility.
* Modal visibility could be mutated by:
  * an action
  * a `link-to` with `(query-params showModal=$VALUE)`
  * calls to
    [`transitionTo`][transitionTo], or
    [`transitionToRoute`][transitionToRoute]
    with the `showModal` query paramter

However, the implementation still had drawbacks:

* Our controller was still responsible for maintaining the visibility flag.
* Our template still had to guard the visibility of the modal with a
  conditional.

Managing modal visibility with nested routes
--------------------------------------------

As an alternative approach to persisting visibility state as a query parameter,
we could instead dedicate a separate route to the modal.

`templates/nested.hbs`:

```handlebars
{% raw %}
{{#link-to "nested.sub"}}
  Open Sub-Route Modal
{{/link-to}}

{{outlet}}

{% endraw %}
```

`templates/nested/sub.hbs`:

```handlebars
{% raw %}
{{#x-modal}}
  {{credit-card card=model}}
{{/x-modal}}
{% endraw %}
```

`routes/nested.js`:

```javascript
App.NestedRoute = Ember.Route.extend({
  actions: {
    closeModal: function() {
      this.transitionTo('nested.index');
    },

    saveCard: function(card) {
      /* save to Stripe here */
      this.transitionTo('nested.index');
    }
  }
});
```

We found this approach to be the best:

* Users could link to, bookmark, navigate away from, and refresh the page while
  maintaining the modal's visibility.
* The modal's visibility could be mutated by:
  * an action
  * a direct `link-to`
  * calls to
    [`transitionTo`][transitionTo], or
    [`transitionToRoute`][transitionToRoute]
* Our controller no longer maintains the visibility flag.
* Our template no longer guards the visibility of the modal with a conditional.

Wrapping up
-----------

Dedicating a sub-route to our payment modal met all of our criteria, however,
depending on the modal's intended behavior, this may not be the case for you.

If the modal is meant to be quick and ephemeral,
use [`Ember.Route#replaceWith`][replaceWith] to close it.

`Ember.Route#replaceWith` removes the current URL from the browser's history
stack, which would prevent access to the modal through the back or refresh
button.

[transitionTo]: http://emberjs.com/api/classes/Ember.Controller.html#method_transitionToRoute
[transitionToRoute]: http://emberjs.com/api/classes/Ember.Route.html#method_transitionTo
[replaceWith]: http://emberjs.com/api/classes/Ember.Route.html#method_replaceWith

[View the live JSBin code below.][jsbin]

<a class="jsbin-embed" href="http://emberjs.jsbin.com/puyude/7/embed?html,js,output">JS Bin</a>
<script src="http://static.jsbin.com/js/embed.js"></script>

[jsbin]: http://emberjs.jsbin.com/puyude/7/edit?html,js,output

The next time you're faced with maintaining application state with an
internal flag, consider embracing the power of the URL.

Additional reading
------------------

* [Ember is for Designers](https://robots.thoughtbot.com/ember-is-for-designers)
* [Ember is for Designers: Alternate States](https://robots.thoughtbot.com/ember-for-designers-alternate-states)
* [Shared Terminology Yet Different Concepts Between Ember.js and Rails](https://robots.thoughtbot.com/shared-terminology-yet-different-concepts-between-emberjs-and-rails)
