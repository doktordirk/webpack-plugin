# aurelia-webpack-plugin

[![npm Version](https://img.shields.io/npm/v/aurelia-webpack-plugin.svg)](https://www.npmjs.com/package/aurelia-webpack-plugin)
[![ZenHub](https://raw.githubusercontent.com/ZenHubIO/support/master/zenhub-badge.png)](https://zenhub.io)
[![Join the chat at https://gitter.im/aurelia/discuss](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/aurelia/discuss?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

This library is part of the [Aurelia](http://www.aurelia.io/) platform and contains a Webpack plugin designed to enable proper Webpack bundling.

> To keep up to date on [Aurelia](http://www.aurelia.io/), please visit and subscribe to [the official blog](http://blog.durandal.io/) and [our email list](http://durandal.us10.list-manage1.com/subscribe?u=dae7661a3872ee02b519f6f29&id=3de6801ccc). We also invite you to [follow us on twitter](https://twitter.com/aureliaeffect). If you have questions, please [join our community on Gitter](https://gitter.im/aurelia/discuss). If you would like to have deeper insight into our development process, please install the [ZenHub](https://zenhub.io) Chrome or Firefox Extension and visit any of our repository's boards. You can get an overview of all Aurelia work by visiting [the framework board](https://github.com/aurelia/framework#boards).

## Installation

Install with npm

```
npm install aurelia-webpack-plugin
```

## Usage

Add the plugin to the webpack config file:

```javascript
var AureliaWebpackPlugin = require('aurelia-webpack-plugin');
var webpackConfig = {
  entry: 'index.js',
  output: {
    path: 'dist',
    filename: 'index_bundle.js'
  },
  plugins: [new AureliaWebpackPlugin()]
}
```

## Configuration options

The plugin accepts an options object with the following properties:

*src*

The directory where the app source files are located. Defaults to './src'.

*root*

The root project directory. Defaults to the directory from where webpack is called.

## Module resource resolution

Module resource resolution is what makes Aurelia's Dependency Injection work. Its what translates require paths to places in the bundle or external files that are loaded asynchronously. 

Some Aurelia modules or plugins have more than 1 file that need to be resolved (for example, when a plugin also contains an html template).
By default, the whole `src` folder their static dependencies are loaded into the Dependency Injection context. 
If you wish to make resources available for DI loader, you need to explicitly `<require>` them either in your `.html` resources or from the `package.json` file. 

Since there are cases that cannot be supported by statically analyzing files (e.g. it is possible to generate require paths dynamically), there's an additional way to declare external dependencies, or in case of Aurelia plugins, to declare internal dependencies by listing those additional resources in the `package.json` file. Listing these dependencies in your package.json allows you include extra files in the bundles.

To see the ways resources can be declared inside `package.json` see the following example:

```js
{
  ...
  "aurelia": {
    "resources": [
      // relative to your /src:
      "some-resource.js",
      "another.html",
      "items/another-without-extension",
      
      // make the file and its dependencies lazy-load from a separate bundle:
      { "path": "external-module/file.html", lazy: true, bundle: "some-bundle" },
      { "path": [], lazy: true, bundle: "some-bundle" },
      
      // include external resource (and its module's dependencies)
      "aurelia-templating-resources/compose"
      
      // include package (and its module's dependencies):
      "bootstrap"
    ],
    
    // you may also override package's root directory in case the file is located at a different place from either the child of main or module's root directory
    "moduleRootOverride": {
      "aurelia-templating-resources": "dist/es2015"
    }
  }
}
```

The Webpack plugin will read that and know to include those files in the bundle.

It'll be intelligent enough to resolve any further dependencies that those files and modules then required statically (regardless if they are JavaScript files, modules with their own `package.json` declarations or additional HTML resources). 
As you can see it is also possible to specify that certain files should be lazy-loaded or bundled separately (if the bundle is specified in the configuration file). To achieve the same directly from templates, add the parameters `bundle="bundle-name"` and/or `lazy` (for lazy-loading) to either the `<require>` tag or to `<compose>` tag, when composing with static names. E.g.:

```html
<require from="./some-file.html" lazy bundle="other-bundle">
```

## Example configuration: custom app directory (other than './src')

```javascript
var path = require('path');
var AureliaWebpackPlugin = require('aurelia-webpack-plugin');
var webpackConfig = {
  entry: 'index.js',
  output: {
    path: 'dist',
    filename: 'index_bundle.js'
  },
  plugins: [
    new AureliaWebpackPlugin({
      src: path.resolve('./app')
    })
  ]
}
```
