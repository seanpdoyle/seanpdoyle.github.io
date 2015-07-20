---
layout: post
title: Migrating FormKeep to ember-cli-rails
modified: 2015-06-18
---

We've recently finished migrating [FormKeep's][formkeep] administrative
dashboard from using the [ember-rails][ember-rails] gem to
[ember-cli-rails][ember-cli-rails].

We chose to migrate to `ember-cli-rails` so that we could
separate our client and server-side codebases and workflows,
align our project's conventions with the Ember.js core team and community,
and utilize the rich [EmberCLI addon ecosystem][ember-addons],
while still running truly end-to-end JavaScript-enabled Capybara integration tests.

[formkeep]: https://formkeep.com/
[ember-rails]: https://github.com/emberjs/ember-rails/
[ember-cli-rails]: https://github.com/rwz/ember-cli-rails/
[ember-addons]: http://www.emberaddons.com/

Our migration consisted of two main steps:

1. Modify our files and directories to be `ember-cli` compliant,
1. Integrate our `ember-cli` app with our Rails app.

## Migrating to `ember-cli`

### Background

Rails applications serving Sprockets-based Ember applications don't enforce any
strict conventions. The location of files is unimportant. Applications are wired
together by global assignments to `window.App`, regardless of the source file's
name or location.

Although it's discouraged, Sprockets-based applications are capable of declaring
classes from any location, sometimes declaring multiple classes per file.
There is nothing preventing `app/assets/javascripts/router.js`
from defining `App.Router` along with `App.PostsRoute` and `App.Post`.

`EmberCLI` applications, on the other hand, use an `ES6` module resolver to
*enforce* that files are named properly and reside in their conventional
location.

The examples in this post will follow the traditional file structure
conventions. If you'd like the migrate to a [pod-based][pods] project,
the approach is similar but the directories are slightly different.

[pods]: http://www.ember-cli.com/#using-pods

### A set of rough guidelines

At a high level, migrations follow a general pattern:

1. Move directories from `app/assets/javascripts/$TYPE` to `app/$TYPE`.
  * `app/assets/javascripts/templates` -> `app/templates`
1. Replace `_` with `-` in all filenames.
  * `app/assets/javascripts/helpers/in_dollars.js` -> `app/helpers/in-dollars.js`
1. Remove `_$TYPE` suffixes.
  * `app/assets/javascripts/views/posts_view.js` -> `app/views/posts.js`
1. Nest children under their parents.
  * `app/assets/javascripts/views/posts_new_view.js` -> `app/views/posts/new.js`

For a more in-depth guide,
check out [this post][ember-rails-to-ember-cli] and
the [ember-cli documentation][ember-cli].

While these guidelines will work for most files, there are some additional
quirks to be aware of.

[ember-cli]: http://www.ember-cli.com/
[ember-rails-to-ember-cli]: http://robots.thoughtbot.com/migrating-from-ember-rails-to-ember-cli

### Ember-Data

Sprockets-based Ember applications typically include an
`app/assets/javascripts/store.js` file responsible for configuring and
extending `Adapters`, `Serializers`, and `Transforms.`

This file commonly ends up becoming a junk drawer full of `ember-data`
configurations.

EmberCLI expects these declarations to be located in
`app/adapters`,
`app/serializers`,
and `app/transforms`,
respectively.

Any calls to `.reopenClass` must be moved to their own initializer.

### Initializers

Sprockets-based applications require initializers to be explicitly registered:

```javascript
// app/assets/javascripts/initializers/foo_initializer.js

Ember.Application.initializer({
  name: "foo",
  initialize: function(container, application)  {
    /* initialize foo */
  }
});
```

In module based EmberCLI applications, the resolver will
[automatically load initializers][ember-cli-initializer]
based on the properties exported by files in `app/initializers`:

```javascript
// app/initializers/foo.js

export default {
  name: "foo",
  initialize: function(container, application)  {
    /* initialize foo */
  }
};
```

[ember-cli-initializer]: http://www.ember-cli.com/#initializers

### Helpers

Sprockets-based applications require helper factories to be registered
by `Ember.Handlebars.helper`:

```javascript
// app/assets/javascripts/helpers/in_dollars_helper.js
Ember.Handlebars.helper("in-dollars", function(amount) {
  /* show in dollars */
});
```

EmberCLI's resolver requires helpers to be created by
`Ember.Handlebars.makeBoundHelper` and exported from their own module:

```javascript
// app/helpers/in-dollars.js

import Ember from "ember";

export default Ember.Handlebars.makeBoundHelper("in-dollars", function(amount) {
 /* show in dollars */
});
```

Special attention must be paid to single-word helpers like `pluralize`.
EmberCLI requires these helpers to be exported as functions, then imported and
registered manually.

```javascript
// helpers/pluralize.js
export default function(word, count) {
  /* pluralize word */
}
```

```javascript
// app/app.js

import Ember from "ember";
import pluralizeHelper from "./helpers/pluralize";

Ember.Handlebars.registerBoundHelper("pluralize", pluralizeHelper);
```

When in doubt, [refer to the documentation][helper-docs].

[ember-cli-helpers]: http://www.ember-cli.com/#helpers
[helper-docs]: http://www.ember-cli.com/#resolving-handlebars-helpers

### Design assets

FormKeep uses the majority of [thoughtbot's design tools][design-tools], which
require SCSS support.

Luckily, there is an Ember Addon for
[SCSS][ember-cli-sass],
[Bourbon][ember-cli-bourbon],
and [Neat][ember-cli-neat]:

```bash
$ ember install ember-cli-{bourbon,neat,sass}
```

When moving styles from `app/assets/stylesheets` to `app/styles`, keep in mind:

* the asset pipeline helpers (such as `image_url`) are `snake_cased`, while
  the EmberCLI tools expect helpers to be `kebab-cased` (such as `image-url`).
* [Node's SCSS compiler][node-sass] doesn't support globbing, so calls to `import directory/*`,
  must be replaced with explicit calls to `import directory/foo`.

[design-tools]: https://thoughtbot.com/open-source#front-end
[ember-cli-sass]: https://www.npmjs.com/package/ember-cli-sass
[ember-cli-bourbon]: https://www.npmjs.com/package/ember-cli-bourbon
[ember-cli-neat]: https://www.npmjs.com/package/ember-cli-neat
[node-sass]: http://www.sitepoint.com/switching-ruby-sass-libsass/

### Tests

Before the migration, FormKeep tested its front end with a combination of
end-to-end JavaScript-enabled [Capybara-Webkit][capybara] feature tests, and
lower-level fine-grained [QUnit BDD][qunit-bdd] tests run with [Teaspoon][teaspoon].

The feature test suite ensured that our front and back ends were wired
together properly, while the JavaScript test suite ensured that our features
behaved the way we intended.

Since our Teaspoon suite was already JavaScript, the migration was very
straightforward:

* move tests from `spec/javascripts/integration` to `tests/acceptance`
* replace `_` with `-` in filenames

For example, `spec/javascripts/integration/user_edits_a_form_spec.js`
became `tests/acceptance/user-edits-a-form-test.js`

Teaspoon's test suites rely on Sprockets' `require` directive to concatenate
auxiliary files onto the global namespace.

```javascript
// spec/javascripts/support/date-helpers.js

window.yesterday = function() { /* get Yesterday's date */ };
window.today = function() { /* get Today's date */ };
```

```javascript
// spec/javascript/spec_helper.js

//= require application
//= require app
//= require support/date-helpers

var App;

window.startApp = function() { /* start the App */ };
window.stopApp = function() { /* stop the App */};

before(function() {
  startApp();
});

after(function() {
  stopApp();
});
```

```javascript
// spec/javascripts/integration/user_creates_form_spec.js

describe("The new form", function() {
  it("is created today", function() {
    // ...
    expect(form.get("createdAt")).to.equal(this.today());
  });
});
```

EmberCLI, on the other hand, avoids declaring values to the global namespace.
Instead, the resolver relies on [ES6 imports][es6-imports] to act as an explicit
declaration of a file's dependencies. In order to invoke a helper method, we
must first `import` it.

```javascript
// tests/helpers/date.js

function yesterday() { /* get Yesterday's date */ }
function today() { /* get Today's date */ }

export {
  yesterday,
  today
};
```

```javascript
// tests/helpers/acceptance.js

function startApp { /* start the App */ }
function stopApp { /* stop the App */}

export {
  startApp,
  stopApp
};
```

```javascript
// tests/acceptance/user-creates-form-test.js

import { startApp, stopApp } from "../helpers/acceptance";
import { today } from "../helpers/date";

describe("The new form", function() {
  before(function() {
    startApp();
  });

  after(function() {
    stopApp();
  });

  it("is created today", function() {
    // ...
    expect(form.get("createdAt")).to.equal(today());
  });
});
```

[capybara]: https://github.com/thoughtbot/capybara-webkit
[qunit-bdd]: https://github.com/square/qunit-bdd
[teaspoon]: https://github.com/modeset/teaspoon
[es6-imports]: http://www.ember-cli.com/#using-modules

### Configuration

Before our migration, we had been declaring application-wide configuration
values (sometimes from Rails' `ENV`) into a `<script>` tag in our document's
`<head>`.

According to EmberCLI, these configuration values should be declared in the
 [environments][environments] file.

[environments]: http://www.ember-cli.com/#Environments

## Integrating with Rails

Once the code that powers our administrative dashboard was separated into its
own EmberCLI app, we used `ember-cli-rails` to integrate with our Rails app.

Our configuration is very similar to the `ember-cli-rails` [getting
started guide][getting-started]:

```ruby
# config/initializers/ember-cli.rb

EmberCLI.configure do |c|
  c.app(
    :frontend,
    build_timeout: 10,
    path: Rails.root.join("frontend"),
    enable: lambda do |path|
      # disable asset compilation during request specs
      !path.starts_with?("/api/")
    end
  )
end
```

[getting-started]: https://github.com/rwz/ember-cli-rails#installation

### Cross Site Request Forgery protection

> Rails' [`protect_from_forgery`](http://api.rubyonrails.org/classes/ActionController/RequestForgeryProtection/ClassMethods.html#method-i-protect_from_forgery) requires a CSRF token for every XHR except for GET.
> The CSRF token is normally found in `app/views/layouts/application.html.*` inserted with the rails helper: [`csrf_meta_tags`](http://api.rubyonrails.org/classes/ActionView/Helpers/CsrfHelper.html#method-i-csrf_meta_tags).

> -- [@tricknotes](https://github.com/emberjs/ember-rails/commit/05ccd1f4c47a80d78ed1ff6ca210a32bc9322b36)

We opted to inject a valid `<meta name="csrf-token">` into our document's
`<head>` before our server rendered the HTML.

On the client side, we added an initializer that introduces a jQuery
`ajaxPrefilter` to add the header:

```javascript
// app/initializers/jquery-csrf.js

export default {
  name: "csrf",
  initialize: function() {
    $.ajaxPrefilter(function(options, originalOptions, xhr) {
      var token = $('meta[name="csrf-token"]').attr("content");
      xhr.setRequestHeader("X-CSRF-Token", token);
    });
  }
};
```

### Workflow

FormKeep's workflow includes running the following in their own processes:

* `foreman start` from the project root runs the API server and serves the
  EmberCLI application with `ember-cli-rails` (through the asset pipeline)
* `ember test --serve` from `app/frontend` runs the `QUnit` tests against
  the front end

Depending upon what is being developed, each of these commands could be run in
isolation.

To run both the Rails and Ember test suite, we declare a `rake` dependency
between `spec` and `ember:test`:

```ruby
# Rakefile

task default: ["spec", "ember:test"]
```

```bash
# run the full test suite
$ rake spec
```

## Our outcome

We're satisfied with where the migration has taken us:

* Our project is aligned with the conventions of the core team and the community
* We're utilizing some awesome Ember addons that wouldn't have been available through Sprockets
* We've taken the first steps towards separating our API from our (one day
  static) client
* Maintained the ability to run truly end-to-end integration tests

## Further readings

When we started our migration, [EmberCLI Migrator][migrator] was in its infancy.
It's a more viable tool now, and definitely worth considering.

[migrator]: https://github.com/fivetanley/ember-cli-migrator
