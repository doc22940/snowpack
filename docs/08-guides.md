## Guides

Snowpack dramatically speeds up your development time by removing the need for a web application bundler. But you can still use build tools like Babel or TypeScript and get the same speed improvements without the bundler. On every change, your build tool will only need to update a single file, instead of entire bundles.

Below are a collection of guides for using different web frameworks and build tools with Snowpack. If you'd like to add your own, feel free to hit the pencil icon in the top-right to edit our docs.

### Babel

To use Babel with Snowpack:

1. Make sure that your entire application lives inside a source directory (ex: `src/`).
2. Run Babel to build your `src/` application to an output `lib/` directory (ex: `babel src/ --out-dir lib --watch`)
3. Update your HTML entrypoint to point to your `lib/` directory.
4. Now make changes to your `src/` directory, and see them build instantly.
5. Optional: Check out our Babel plugin for importing packages by name in the guide above.


### TypeScript

TypeScript expects imports to be by package name, and won't be able to understand your "web_modules/" imports by default. You'll need to follow one of the following guides based on your setup:

#### With Babel (Recommended)

If you build your site with Babel, you can keep importing packages by name thanks to our [Snowpack Plugin](#importing-packages-by-name). TypeScript will be able to understand these "package name imports" and properly map them back to the type definitions found in your `node_modules/` directory when it type checks your source directory.

To use TypeScript with Babel, [set up Babel](#babel) with the following plugins (feel free to add others):

```js
/* .babelrc */
{
  "presets": ["@babel/preset-typescript"],
  "plugins": ["snowpack/assets/babel-plugin.js"]
}
```
 
Once you have Babel building your site correctly, TypeScript should be able to understand your source directory without any additional configuration.


#### Without Babel:

To use TypeScript with Snowpack, you'll need to set up your tsconfig.json to understand "web_module" import paths. See our annotated tsconfig.json example below:

```js
"compilerOptions": {
  // Choose your target based on which browsers you'd like to support.
  "target": "es2017",
  // Required: Use module="esnext" so that TS won't compile/disallow any ESM syntax.
  "module": "esnext",
  // Optional, but recommended
  "moduleResolution": "node",
  // Optional, but recommended
  "baseUrl": ".",
  // Required: Map "/web_modules/*" imports back to their node_modules/ TS definition files.
  "paths": {
      "/web_modules/*.js": [
          "node_modules/@types/*",
          "node_modules/*",
          "web_modules/*.js"
      ]
  },
  // ...
}
```

### Vue

```js
/*
 * NOTE: The Vue package points to the runtime-only distribution by default.
 * Unless you are using the Vue CLI, you'll most likely need the full browser build.
 * Runtime only: `import Vue from "/web_modules/vue.js"`
 *
 * https://vuejs.org/v2/guide/installation.html#Explanation-of-Different-Builds
 */
import Vue from "/web_modules/vue/dist/vue.esm.browser.js";

// $ snowpack --include "src/index.js"
// ✔ snowpack installed: vue/dist/vue.esm.browser.js. [1.07s]
```

> Psst... are you an expert on Vue? We'd [love your help](https://github.com/pikapkg/snowpack/blob/master/docs) writing a short guide for authoring `.vue` SFC's and then compiling them to valid JS!

### Preact

```js
// File: src/index.js
import { h, Component, render } from '/web_modules/preact.js';
// Optional, if using HTM instead of JSX and Babel:
import htm from '/web_modules/htm.js';
const html = htm.bind(h);

// $ snowpack --include "src/index.js"
// ✔ snowpack installed: preact, htm. [1.06s]
```

#### Relevant guides:
- [Quick Start](#quick-start) (Written for Preact!)
- [Snowpack + Babel](#babel) (Required for JSX)
- [HTM](#htm) (Alternative to JSX)
- [Importing Packages by Name](#importing-packages-by-name) (Optional)


### React

<p>
<iframe width="560" height="315" src="https://www.youtube.com/embed/pUUAil_9yIw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

```js
import React, { useState } from '/web_modules/react.js';
```

**Important:** React is [not yet published with ES Module support](https://github.com/facebook/react/issues/11503), and the way that it's written makes it impossible to install as a dependency (*"Error: '__moduleExports' is not exported by node_modules/react/index.js"*). **However**, React is still supported thanks to our actively-maintained ESM builds of [React](https://www.npmjs.com/package/@pika/react) & [React-DOM](https://www.npmjs.com/package/@pika/react). You can install them both in your project under an alias like so:

```
npm install react@npm:@pika/react react-dom@npm:@pika/react-dom
   yarn add react@npm:@pika/react react-dom@npm:@pika/react-dom
```

When installed under the react/react-dom alias, all tooling (including Snowpack) will benefit from these easier-to-analyze ESM distributions of the popular packages.




### JSX

It's important to keep in mind that JSX isn't really JavaScript. JSX is a build-time syntax that only your build tooling understands, and that doesn't run directly in any browser. In any app you work on (bundled or unbundled) you'll need to use a build tool like Babel or TypeScript to transpile JSX into regular old JavaScript before shipping it to the browser.

To use JSX with Snowpack, you can either:

1. Use TypeScript with ["--jsx" mode](https://www.typescriptlang.org/docs/handbook/jsx.html) enabled. See our "TypeScript" guide for more.
1. Use Babel with a plugin like [@babel/plugin-transform-react-jsx](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx) or a framework-specific preset like [@babel/preset-react](https://babeljs.io/docs/en/babel-preset-react). See our "Babel" guide for more.
1. Use a JSX-like alternative that can run in the browser. Jason Miller's [htm](https://github.com/developit/htm) is a great option (keep reading).


### HTM

HTM is "a JSX alternative using [...] JSX-like syntax in plain JavaScript - no transpiler necessary." It has first-class support for Preact (built by the same team) but also works with React.

https://www.pika.dev happily used HTM + Preact for many months before adding Babel and switching to React + TypeScript.

```js
// File: src/index.js
import { h, Component, render } from '/web_modules/preact.js';
import htm from '/web_modules/htm.js';
const html = htm.bind(h);

return html`<div id="foo" foo=${40 + 2}>Hello!</div>`

// $ snowpack --include "src/index.js"
// ✔ snowpack installed: preact, htm. [1.06s]
```

### lit-html

[lit-html](https://lit-html.polymer-project.org/) is "an efficient, expressive, extensible HTML templating library for JavaScript." Similarly to [HTM](#htm), lit-html uses tagged template literals for JSX-like syntax in the browser without requiring any transpilation.

```js
// File: src/index.js
import { html } from "/web_modules/lit-html.js";
import { until } from "/web_modules/lit-html/directives/until.js";

// $ snowpack --include "src/index.js"
// ✔ snowpack installed: lit-html, lit-html/directives/until.js [0.17s]
```

**Important:** There should only ever be one version of lit-html run in the browser. If you are using third-party packages that also depend on lit-html, we strongly recommend adding `"lit-html"` to your [dedupe](#all-config-options) config to prevent Snowpack from installing multiple versions:

```js
// File: package.json
  "snowpack": {
    // ...
    "dedupe": ["lit-html"]
  },
```

**Important:** [lit-html directives](https://lit-html.polymer-project.org/guide/template-reference#built-in-directives) aren't exported by the main package. Run Snowpack with the `--include` flag so that Snowpack can automatically detect these imports in your application. Otherwise, add them each separately to the `webDependencies` whitelist in your `package.json`:

```js
// File: package.json
  "snowpack": {
    "webDependencies": [
      "lit-html",
      "lit-html/directives/until.js",
    ],
  },
```


### LitElement

[LitElement](https://lit-element.polymer-project.org/) is "a simple base class for creating fast, lightweight web components" built on top of [lit-html](https://lit-html.polymer-project.org/). Read our [lit-html](#lit-html) guide for important information.

```js
// File: src/index.js
import { LitElement, html, css } from "/web_modules/lit-element.js";
import { repeat } from "/web_modules/lit-html/directives/repeat.js";

// $ snowpack --include "src/index.js"
// ✔ snowpack installed: lit-html, lit-element [0.25s]
```

### Svelte

[Svelvet](https://github.com/jakedeichert/svelvet) is a [Svelte](https://github.com/sveltejs/svelte) compiler + dev environment that integrates directly with Snowpack. Svelvet watches your source directory and compiles your Svelte components to standard, browser-ready JavaScript.

```html
<!-- File: src/App.svelte -->
<!-- Taken from example app: https://github.com/jakedeichert/svelvet/tree/master/examples/basic -->
<script>
    import Header from './components/Header';
    import Footer from './components/Footer/index';
</script>
<Header />
<Footer />
```

Run `npx svelvet` on the command line to start Svelvet in dev mode. For more information on Svelvet and instructions on how to build for production, [check out the full guide](https://github.com/jakedeichert/svelvet).


### SASS

```toml
# 1. Build your CSS (add --watch to watch for changes)
sass ./scss:./css
# 2. Load the output css anywhere in your application
<link rel="stylesheet" type="text/css" href="/css/index.css">
```

[SASS](https://sass-lang.com/) is a "mature, stable, and powerful professional grade CSS extension language" that compiles to CSS. Use the [SASS CLI](https://sass-lang.com/install) to compile your SASS into CSS, and then load that CSS anywhere in your application.

**You might also like:** [csz](https://github.com/lukejacksonn/csz), run-time CSS modules with support for SASS-like selectors. CSZ runs directly in the browser, so you can skip the SASS build/watch step. 

 
### Tailwind CSS

```toml
# 1. Build your CSS
npx tailwind build styles.css -o css/tailwind.css
# 2. Load the output file anywhere in your application
<link rel="stylesheet" type="text/css" href="/css/tailwind.css">
```

Tailwind works as-expected with Snowpack. Just follow [the official Tailwind Install Guide](https://tailwindcss.com/docs/installation/). When you get to step #4 ("Process your CSS with Tailwind") choose the official Tailwind CLI option to generate your CSS. Import that generated CSS file in your HTML application.


### Styled Components

```js
// File: src/index.js
import React from "/web_modules/react.js";
import {render} from "/web_modules/react-dom.js";
import styled from "/web_modules/styled-components.js";

// $ snowpack --include "src/index.js"
// ✔ snowpack installed: react, react-dom, styled-components. [1.06s]
```

#### Relevant guides:
- [Snowpack + React](#React) (Required)
- [Snowpack + Babel](#babel) (Required for JSX)
- [Importing Packages by Name](#importing-packages-by-name) (Optional)


### Material UI

```js
// File: src/index.js
import React from "/web_modules/react.js";
import {render} from "/web_modules/react-dom.js";
import Button from "/web_modules/@material-ui/core/Button/index.js";

// $ snowpack --include "src/index.js"
// ✔ snowpack installed: @material-ui/core/Button/index.js, react, react-dom. [1.64s]
```

#### Relevant guides:
- [Snowpack + React](#React) (Required)
- [Snowpack + Babel](#babel) (Required for JSX)
- [Importing Packages by Name](#importing-packages-by-name) (Optional)

### Workbox

The [Workbox CLI](https://developers.google.com/web/tools/workbox/modules/workbox-cli) integrates well with Snowpack. Run the wizard to bootstrap your first configuration file, and then run `workbox generateSW` to generate your service worker.

Remember that Workbox expects to be run every time you deploy, as a part of a production "build" process (similar to how Snowpack's [`--optimize`](#production-optimization) flag works). If you don't have one yet, create a [`"deploy"` and/or `"build"` script](https://michael-kuehnel.de/tooling/2018/03/22/helpers-and-tips-for-npm-run-scripts.html) in your `package.json` to automate your production build process.


### Migrating an Existing App

How you migrate an existing app to Snowpack depends on which Bundler features/plugins you're using. If you're only using the `import` statement to import other JavaScript files, the process should only take a couple of minutes. If you're importing CSS, images, or other non-JS content in your application, you'll need to first get rid of those Webpack-specific imports before migrating away from Webpack.

Assuming you've removed all code specific to your bundler, you can use the following rough plan to migrate to Snowpack.

1. Use Babel to assist in the migration. If you don't want to use Babel, don't worry; You can always remove it after migrating.
1. Follow the Babel guide above to build your existing `src/` directory to a new `lib/` directory.
1. Follow the Babel Plugin guide above to add the Snowpack Babel plugin so that your package imports will continue to run as is. Check your output `lib/` directory to make sure that dependency imports are being rewritten as expected.
1. Run your application in the browser! If everything is working, you're done! Otherwise, use your browser's dev tools to hunt down any remaining issues in your application code.

### Migrating to Webpack/Parcel

Snowpack is designed for zero lock-in. If you ever feel the need to add a traditional application bundler to your stack (for whatever reason!) you can do so in seconds.

Any application built with Snowpack should Just Work™️ when passed through Webpack/Rollup/Parcel. If you are importing packages by full URL (ex: `import React from '/web_modules/react.js'`), then a simple Find & Replace should help you re-write them to the plain package names  (ex: `import React from 'react'`) that bundlers expect.

