EMBER JS
========

Install and Create an App
=========================

```sh
$ npm install -g ember-cli

$ ember new my-app --lang en  # setting lang improves the generated app's accesibility
```

The generated app has the following structure:

```
my-app
├── app
│   ├── components
│   ├── controllers
│   ├── helpers
│   ├── models
│   ├── routes
│   ├── styles                  # .css files
│   ├── templates               # .hbs files 
│   ├── app.js
│   ├── index.html
│   └── router.js
├── config
│   ├── ember-cli-update.json
│   ├── environment.js
│   ├── optional-features.json
│   └── targets.js
├── public
│   └── robots.txt
├── tests
│   ├── helpers
│   │   └── index.js
│   ├── integration
│   ├── unit
│   ├── index.html
│   └── test-helper.js
├── vendor
├── [ some dotfiles... ]
├── README.md
├── ember-cli-build.js
├── package.json
├── package-lock.json
└── testem.js
```

Development
===========

The dev server compiles assets, serves the content on `http://localhost:4200/` and live-reloads the app on change.

```sh
$ ember server
```

Router
======

From Ember's tutorial:

`app/router.js`
```js
import EmberRouter from '@ember/routing/router';
import config from 'super-rentals/config/environment';

export default class Router extends EmberRouter {
  location = config.locationType;
  rootURL = config.rootURL;
}

Router.map(function () {
  this.route('about');      // Route path defaults to `/route-name`, here `/about`
  this.route('contact', { path: '/getting-in-touch' });
  this.route('rental', { path: '/rentals/:rental_id' });  // `:param` is then available as TODO
});
```

The router searches **top to bottom** and stop at first match. Path specification is actually an expression TODO: which type?

After the route is defines, on hit, Ember automatically:
- Renders the template named after it (`app/templates/route-name.hbs`)
- 

Regular `<a>` links work but trigger refreshes. To use the SPA functionality use the `<LinkTo> @route="route-name" />` component (see below).

Templates
=========

The router renders the template named after the matched route automatically, searching the `app/templates` dir. Then, nested components are rendered using the templates (file)named after them, searching the `app/components` dir.

TODO:

`{{#if ...}}...{{else}}...{{/if}}`

Application Template
--------------------

Named `application.hbs`, acts as a layout for all pages. Then `{{outlet}}` is replaced by the route template content.

From Ember's tutorial:

```hbs
<div class="container">
  <NavBar />
  <div class="body">
    {{outlet}}
  </div>
</div>
```

Components
==========

Their templates live in `app/components`. A template is enough to provide a simple component without behavior. To code behavior, just add (or generate) a JS file to the same dir, following the naming conventions: `component-name[.hbs|.js]` <-> `class ComponentNameComponent` <-> `<ComponentName />`.

In the template, `{{yield}}` is replaced by the inner HTML of the invocation tag.

A minimal example adapted from Ember's tutorial:

`app/components/jumbo.hbs`:
```hbs
<div class="jumbo">
  <div class="right tomster"></div>
  {{yield}}
</div>
```

Invocation:
```hbs
<Jumbo>
  <h2>Welcome to Super Rentals!</h2>
  ...
</Jumbo>
```

But there's much more to components:

`...attributes` is used in component templates to capture any HTML attributes passed in the invocation, like `alt` in the example below, so they can be applied to a (normally protagonic) element in the component template, below the `<img>` tag. Note order is relevant when using `...attributes` within other HTML attrs, as later ones override earlier ones.

On invocation, components can also receive `@argument`s that can be used in the template as `{{@argument}}`s. If the component has JS behavior, these can be accessed. Glimmer classes expose them within `this.args`.

State is managed using instance vars of the controller class. To make Ember track changes, add the `@tracked` decorator to the var, as in `@tracked isLarge = false`.

Actions are methods accessible from the template. Just regular functions decorated with `@action`, as in `@action toggleSize() { ... }`.

You can define setters as `set property(assigned_value) { this.property = ... }` that are executed like `property = 3`, and getters as `get property() { return ... }` that are executed like `x.property`.

The `on` helper, used as JS like in `{{on "click" this.toggleSize}}`, is used in the template to attach event handlers.

This is a more complete example. Also adapted from Ember's tutorial, mixing in the sizing behavior of rental/image:

`app/components/map.hbs`:
```js
import Component from '@glimmer/component';  // There are also classic components, etc.
import { tracked } from '@glimmer/tracking';
import { action } from '@ember/object';

import ENV from 'super-rentals/config/environment';  // See about envs below.

const MAPBOX_API = 'https://api.mapbox.com/styles/v1/mapbox/streets-v11/static';

export default class MapComponent extends Component {

  //// Sizing behavior ////

  @tracked isLarge = false;

  @action toggleSize() {
    this.isLarge = !this.isLarge;
  }

  get dimensions() {
    if this.isLarge {
      width="150";
      height="150";
    } else {
      width="400";
      height="400";
    }

    return { width, height }
  }

  //// Map src generation ////

  get src() {
    // `this.args` encapsulate all `@argument`s used in the invocation
    let { lng, lat, zoom } = this.args;
    let { width, height } = this.dimensions;

    let coordinates = `${lng},${lat},${zoom}`;
    let dimensions = `${width}x${height}`;
    let accessToken = `access_token=${encodeURIComponent(ENV.MAPBOX_ACCESS_TOKEN)}`;

    return `${MAPBOX_API}/${coordinates}/${dimensions}@2x?${accessToken}`;
  }
}
```

`app/components/map.hbs`:
```hbs
<div class="map">
  <img
    alt="Map image at coordinates {{@lat}},{{@lng}}"
    ...attributes
    src={{this.src}}
    width={{this.dimensions.width}}
    height={{this.dimensions.height}}
  >
  <a {{on "click" this.toggleSize}}>
    View {{if this.isLarge "Smaller" "Larger"}}
  </a>
</div>
```

Invocation (assuming a `rental` dict exists):
```hbs
<Map
  @lat={{rental.location.lat}}
  @lng={{rental.location.lng}}
  @zoom="9"
  alt="A map of {{rental.title}}"
/>
```

About setting state in components
---------------------------------

TODO

Generate with Ember CLI
-----------------------

```sh
$ ember generate component [--with-component-class] <component-name>
$ ember generate component-class <component-name>  # Only the JS class
```

The component name can be namespaced with `/` separators. This generate files in the correct folders and use `::` in names, so: `app/components/rental/image[.hbs|.js]` <-> `class RentalImageComponent` <-> `<Rental::Image />`.
