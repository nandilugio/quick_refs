F R O N T E N D   S T A C K
===========================

JS Package Managers
===================

npm
---

- Included in node.js.
- Uses the `package.json` file to:
  - Specify project metadata (to publish in npm repo).
  - Specify `dependencies` and `devDependencies`.
  - Specify simple one-liner task automation eg. compile, lint, test...
- Generates a lock file for installed packages named `package-lock.json`.

Yarn (1 or Classic)
-------------------

- More modern alternative to npm with simpler implementation.
- Uses npm repos.
- Generates a lock file for installed packages named `yarn.lock`.

TODO: newer versions

Compilers & Preprocessors -> JS
===============================

Babel
-----

- Compiles modern JS to older, more cross-browser compatible JS.
- It's configured using the `babel.config.js` file.
- Uses `presets` to target older browsers or JS specifications.
  - `babel-preset-env` is a preset that is meant to generate compatible JS from latest gen JS. It integrates other tools like `browserslist`, `compat-table`, and `electron-to-chromium`. 
- Is customizable using `plugins`.

TypeScript
----------

Elm
---

CSS Preprocessors
=================

Sass
----

Asset pipelines
===============

Rails: Sprockets
----------------

- All assets are served by rails from directories configured on the _assets path_. When serving, Sprockets:
  - Checks if there are directives eg. including other files in
  - Checks the (last) extension in the filename and preprocess the file accordingly, eg. with Sass or TypeScript
- Assets are then included in view templates using special helpers that know the public path of those assets.
- Given the above, you can create bundles just by creating _manifest_ files: regular files that just reference (include) other files. You can normally fing files like `application.js`, referencing all js in the app, or the share used in all views, or at least in the main layout.

Webpack
-------

Provides pack (code bundles) generation, in JS implementing the modules API (`import from`, `export`, etc.). It also provides preprocessing of the files through `loaders`, which for JS include `babel-loader` for Babel, `ts-loader` to compile TypeScript, etc. For CSS include `css-loader` and `style-loader` to compose and inject regular CSS, etc.

It's configured using the `webpack.config.js`, which specifies things like:
  - the `entry` file that will be used as a start for processing and inclusion
  - a JS `target` spec (eg. `es5`) for its generated code (eg. code needed for the module API)
  - a set of `module.rules` that define the loaders to `use` for filenames matching a regex `test` (eg. `{ test: /\.js$/; use: 'babel-loader' }`).

### Rails: Webpacker



Code Quality Tools
==================
