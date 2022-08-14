# Webpack Guide


<img src="./Webpack Guide.assets/logo-on-white-bg.png" alt="Branding Guidelines | webpack"  />



## 0. Introduction

<img src="./Webpack Guide.assets/BudleYourJS.jpg"  />



## 1. Basic Usage

### 1.1 Getting Started

Webpack is used to compile JavaScript modules. Once [installed](https://webpack.js.org/guides/installation), you can interact with webpack either from its [CLI](https://webpack.js.org/api/cli) or [API](https://webpack.js.org/api/node). If you're still new to webpack, please read through the [core concepts](https://webpack.js.org/concepts) and [this comparison](https://webpack.js.org/comparison) to learn why you might use it over the other tools that are out in the community.

> :bangbang: The minimum supported Node.js version to run webpack 5 is 10.13.0 (LTS)

#### 1.1.1 Basic Setup

First let's create a directory, initialize npm, [install webpack locally](https://webpack.js.org/guides/installation/#local-installation), and install the [`webpack-cli`](https://github.com/webpack/webpack-cli) (the tool used to run webpack on the command line):

```bash
mkdir webpack-demo
cd webpack-demo
npm init -y
npm install webpack webpack-cli --save-dev
```

Throughout the Guides we will use **`diff`** blocks to show you what changes we're making to directories, files, and code. For instance:

```diff
+ this is a new line you shall copy into your code
- and this is a line to be removed from your code
  and this is a line not to touch.
```

Now we'll create the following directory structure, files and their contents:

**project**

```diff
  webpack-demo
  |- package.json
  |- package-lock.json
+ |- index.html
+ |- /src
+   |- index.js
```

**src/index.js**

```javascript
function component() {
  const element = document.createElement('div');

  // Lodash, currently included via a script, is required for this line to work
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

**index.html**

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Getting Started</title>
    <script src="https://unpkg.com/lodash@4.17.20"></script>
  </head>
  <body>
    <script src="./src/index.js"></script>
  </body>
</html>
```

We also need to adjust our `package.json` file in order to make sure we mark our package as `private`, as well as removing the `main` entry. This is to prevent an accidental publish of your code.

> :o:If you want to learn more about the inner workings of `package.json`, then we recommend reading the [npm documentation](https://docs.npmjs.com/files/package.json).
>

**package.json**

```diff
 {
   "name": "webpack-demo",
   "version": "1.0.0",
   "description": "",
-  "main": "index.js",
+  "private": true,
   "scripts": {
     "test": "echo \"Error: no test specified\" && exit 1"
   },
   "keywords": [],
   "author": "",
   "license": "MIT",
   "devDependencies": {
     "webpack": "^5.38.1",
     "webpack-cli": "^4.7.2"
   }
 }
```

In this example, there are implicit dependencies between the `<script>` tags. Our `index.js` file depends on `lodash` being included in the page before it runs. This is because `index.js` never explicitly declared a need for `lodash`; it assumes that the global variable `_` exists.

There are problems with managing JavaScript projects this way:

- It is not immediately apparent that the script depends on an external library.
- If a dependency is missing, or included in the wrong order, the application will not function properly.
- If a dependency is included but not used, the browser will be forced to download unnecessary code.

Let's use webpack to manage these scripts instead.

#### 1.1.2 Creating a Bundle

First we'll tweak our directory structure slightly, separating the "source" code (`./src`) from our "distribution" code (`./dist`). The "source" code is the code that we'll write and edit. The "distribution" code is the minimized and optimized `output` of our build process that will eventually be loaded in the browser. Tweak the directory structure as follows:

**project**

```diff
  webpack-demo
  |- package.json
  |- package-lock.json
+ |- /dist
+   |- index.html
- |- index.html
  |- /src
    |- index.js
```

> :o:You may have noticed that `index.html` was created manually, even though it is now placed in the `dist` directory. Later on in [another guide](https://webpack.js.org/guides/output-management/#setting-up-htmlwebpackplugin), we will generate `index.html` rather than edit it manually. Once this is done, it should be safe to empty the `dist` directory and to regenerate all the files within it.
>

To bundle the `lodash` dependency with `index.js`, we'll need to install the library locally:

```bash
npm install --save lodash
```

> :o:When installing a package that will be bundled into your production bundle, you should use `npm install --save`. If you're installing a package for development purposes (e.g. a linter, testing libraries, etc.) then you should use `npm install --save-dev`. More information can be found in the [npm documentation](https://docs.npmjs.com/cli/install).
>

Now, let's import `lodash` in our script:

**src/index.js**

```diff
+import _ from 'lodash';
+
 function component() {
   const element = document.createElement('div');

-  // Lodash, currently included via a script, is required for this line to work
+  // Lodash, now imported by this script
   element.innerHTML = _.join(['Hello', 'webpack'], ' ');

   return element;
 }

 document.body.appendChild(component());
```

Now, since we'll be bundling our scripts, we have to update our `index.html` file. Let's remove the lodash `<script>`, as we now `import` it, and modify the other `<script>` tag to load the bundle, instead of the raw `./src` file:

**dist/index.html**

```diff
 <!DOCTYPE html>
 <html>
   <head>
     <meta charset="utf-8" />
     <title>Getting Started</title>
-    <script src="https://unpkg.com/lodash@4.17.20"></script>
   </head>
   <body>
-    <script src="./src/index.js"></script>
+    <script src="main.js"></script>
   </body>
 </html>
```

In this setup, `index.js` explicitly requires `lodash` to be present, and binds it as `_` (no global scope pollution). By stating what dependencies a module needs, webpack can use this information to build a dependency graph. It then uses the graph to generate an optimized bundle where scripts will be executed in the correct order.

With that said, let's run `npx webpack`, which will take our script at `src/index.js` as the [entry point](https://webpack.js.org/concepts/entry-points), and will generate `dist/main.js` as the [output](https://webpack.js.org/concepts/output). The `npx` command, which ships with Node 8.2/npm 5.2.0 or higher, runs the webpack binary (`./node_modules/.bin/webpack`) of the webpack package we installed in the beginning:

```bash
$ npx webpack
[webpack-cli] Compilation finished
asset main.js 69.3 KiB [emitted] [minimized] (name: main) 1 related asset
runtime modules 1000 bytes 5 modules
cacheable modules 530 KiB
  ./src/index.js 257 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 1851 ms
```

> :o:Your output may vary a bit, but if the build is successful then you are good to go. 
>

Open `index.html` from the `dist` directory in your browser and, if everything went right, you should see the following text: `'Hello webpack'`.

#### 1.1.3 Modules

The [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) and [`export`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) statements have been standardized in [ES2015](https://babeljs.io/docs/en/learn/). They are supported in most of the browsers at this moment, however there are some browsers that don't recognize the new syntax. But don't worry, webpack does support them out of the box.

Behind the scenes, webpack actually "**transpiles**" the code so that older browsers can also run it. If you inspect `dist/main.js`, you might be able to see how webpack does this, it's quite ingenious! Besides `import` and `export`, webpack supports various other module syntaxes as well, see [Module API](https://webpack.js.org/api/module-methods) for more information.

Note that webpack will not alter any code other than `import` and `export` statements. If you are using other [ES2015 features](http://es6-features.org/), make sure to [use a transpiler](https://webpack.js.org/loaders/#transpiling) such as [Babel](https://babeljs.io/) via webpack's [loader system](https://webpack.js.org/concepts/loaders/).

#### 1.1.4 Using a Configuration

As of version 4, webpack doesn't require any configuration, but most projects will need a more complex setup, which is why webpack supports a [configuration file](https://webpack.js.org/concepts/configuration). This is much more efficient than having to manually type in a lot of commands in the terminal, so let's create one:

**project**

```diff
  webpack-demo
  |- package.json
  |- package-lock.json
+ |- webpack.config.js
  |- /dist
    |- index.html
  |- /src
    |- index.js
```

**webpack.config.js**

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

Now, let's run the build again but instead using our new configuration file:

```bash
$ npx webpack --config webpack.config.js
[webpack-cli] Compilation finished
asset main.js 69.3 KiB [compared for emit] [minimized] (name: main) 1 related asset
runtime modules 1000 bytes 5 modules
cacheable modules 530 KiB
  ./src/index.js 257 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 1934 ms
```

> :o:If a `webpack.config.js` is present, the `webpack` command picks it up by default. We use the `--config` option here only to show that you can pass a configuration of any name. This will be useful for more complex configurations that need to be split into multiple files.

A configuration file allows far more flexibility than CLI usage. We can specify loader rules, plugins, resolve options and many other enhancements this way. See the [configuration documentation](https://webpack.js.org/configuration) to learn more.

#### 1.1.5 NPM Scripts

Given it's not particularly fun to run a local copy of webpack from the CLI, we can set up a little shortcut. Let's adjust our *package.json* by adding an [npm script](https://docs.npmjs.com/misc/scripts):

**package.json**

```diff
 {
   "name": "webpack-demo",
   "version": "1.0.0",
   "description": "",
   "private": true,
   "scripts": {
-    "test": "echo \"Error: no test specified\" && exit 1"
+    "test": "echo \"Error: no test specified\" && exit 1",
+    "build": "webpack"
   },
   "keywords": [],
   "author": "",
   "license": "ISC",
   "devDependencies": {
     "webpack": "^5.4.0",
     "webpack-cli": "^4.2.0"
   },
   "dependencies": {
     "lodash": "^4.17.20"
   }
 }
```

Now the `npm run build` command can be used in place of the `npx` command we used earlier. Note that within `scripts` we can reference locally installed npm packages by name the same way we did with `npx`. This convention is the standard in most npm-based projects because it allows all contributors to use the same set of common scripts.

Now run the following command and see if your script alias works:

```bash
$ npm run build

...

[webpack-cli] Compilation finished
asset main.js 69.3 KiB [compared for emit] [minimized] (name: main) 1 related asset
runtime modules 1000 bytes 5 modules
cacheable modules 530 KiB
  ./src/index.js 257 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 1940 ms
```

###### Tip

Custom parameters can be passed to webpack by adding two dashes between the `npm run build` command and your parameters, e.g. `npm run build -- --color`.

#### 1.1.6 Conclusion

Now that you have a basic build together you should move on to the next guide [`Asset Management`](https://webpack.js.org/guides/asset-management) to learn how to manage assets like images and fonts with webpack. At this point, your project should look like this:

**project**

```txt
webpack-demo
|- package.json
|- package-lock.json
|- webpack.config.js
|- /dist
  |- main.js
  |- index.html
|- /src
  |- index.js
|- /node_modules
```

###### Warning

Do not compile untrusted code with webpack. It could lead to execution of malicious code on your computer, remote servers, or in the Web browsers of the end users of your application.

If you want to learn more about webpack's design, you can check out the [basic concepts](https://webpack.js.org/concepts) and [configuration](https://webpack.js.org/configuration) pages. Furthermore, the [API](https://webpack.js.org/api) section digs into the various interfaces webpack offers.

## 2. Asset Management

If you've been following the guides from the start, you will now have a small project that shows "Hello webpack". Now let's try to incorporate some other assets, like images, to see how they can be handled.

Prior to webpack, front-end developers would use tools like [grunt](https://gruntjs.com/) and [gulp](https://gulpjs.com/) to process these assets and move them from their `/src` folder into their `/dist` or `/build` directory. The same idea was used for JavaScript modules, but tools like webpack will **dynamically bundle** all dependencies (creating what's known as a [dependency graph](https://webpack.js.org/concepts/dependency-graph)). This is great because every module now *explicitly states its dependencies* and we'll avoid bundling modules that aren't in use.

One of the coolest webpack features is that you can also *include any other type of file*, besides JavaScript, for which there is a loader or built-in [Asset Modules](https://webpack.js.org/guides/asset-modules/) support. This means that the same benefits listed above for JavaScript (e.g. explicit dependencies) can be applied to everything used in building a website or web app. Let's start with CSS, as you may already be familiar with that setup.

### 2.1 Setup

Let's make a minor change to our project before we get started:

**dist/index.html**

```diff
 <!DOCTYPE html>
 <html>
   <head>
     <meta charset="utf-8" />
-    <title>Getting Started</title>
+    <title>Asset Management</title>
   </head>
   <body>
-    <script src="main.js"></script>
+    <script src="bundle.js"></script>
   </body>
 </html>
```

**webpack.config.js**

```diff
 const path = require('path');

 module.exports = {
   entry: './src/index.js',
   output: {
-    filename: 'main.js',
+    filename: 'bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
 };
```

### 2.2 Loading CSS

In order to `import` a CSS file from within a JavaScript module, you need to install and add the [style-loader](https://webpack.js.org/loaders/style-loader) and [css-loader](https://webpack.js.org/loaders/css-loader) to your [`module` configuration](https://webpack.js.org/configuration/module):

```bash
npm install --save-dev style-loader css-loader
```

**webpack.config.js**

```diff
 const path = require('path');

 module.exports = {
   entry: './src/index.js',
   output: {
     filename: 'bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
+  module: {
+    rules: [
+      {
+        test: /\.css$/i,
+        use: ['style-loader', 'css-loader'],
+      },
+    ],
+  },
 };
```

Module loaders can be chained. Each loader in the chain applies transformations to the processed resource. A chain is executed in reverse order. The first loader passes its result (resource with applied transformations) to the next one, and so forth. Finally, webpack expects JavaScript to be returned by the last loader in the chain.

The above order of loaders should be maintained: [`'style-loader'`](https://webpack.js.org/loaders/style-loader) comes first and followed by [`'css-loader'`](https://webpack.js.org/loaders/css-loader). If this convention is not followed, webpack is likely to throw errors.

> :o:webpack uses a regular expression to determine which files it should look for and serve to a specific loader. In this case, any file that ends with `.css` will be served to the `style-loader` and the `css-loader`. 

This enables you to `import './style.css'` into the file that depends on that styling. Now, when that module is run, a `<style>` tag with the stringified css will be inserted into the `<head>` of your html file.

Let's try it out by adding a new `style.css` file to our project and import it in our `index.js`:

**project**

```diff
  webpack-demo
  |- package.json
  |- package-lock.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
+   |- style.css
    |- index.js
  |- /node_modules
```

**src/style.css**

```css
.hello {
  color: red;
}
```

**src/index.js**

```diff
 import _ from 'lodash';
+import './style.css';

 function component() {
   const element = document.createElement('div');

   // Lodash, now imported by this script
   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+  element.classList.add('hello');

   return element;
 }

 document.body.appendChild(component());
```

Now run your build command:

```bash
$ npm run build

...
[webpack-cli] Compilation finished
asset bundle.js 72.6 KiB [emitted] [minimized] (name: main) 1 related asset
runtime modules 1000 bytes 5 modules
orphan modules 326 bytes [orphan] 1 module
cacheable modules 539 KiB
  modules by path ./node_modules/ 538 KiB
    ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
    ./node_modules/style-loader/dist/runtime/injectStylesIntoStyleTag.js 6.67 KiB [built] [code generated]
    ./node_modules/css-loader/dist/runtime/api.js 1.57 KiB [built] [code generated]
  modules by path ./src/ 965 bytes
    ./src/index.js + 1 modules 639 bytes [built] [code generated]
    ./node_modules/css-loader/dist/cjs.js!./src/style.css 326 bytes [built] [code generated]
webpack 5.4.0 compiled successfully in 2231 ms
```

Open up `dist/index.html` in your browser again and you should see that `Hello webpack` is now styled in red. To see what webpack did, inspect the page (don't view the page source, as it won't show you the result, because the `<style>` tag is dynamically created by JavaScript) and look at the page's head tags. It should contain the style block that we imported in `index.js`.

Note that you can, and in most cases should, [minimize css](https://webpack.js.org/plugins/mini-css-extract-plugin/#minimizing-for-production) for better load times in production. On top of that, loaders exist for pretty much any flavor of CSS you can think of – [postcss](https://webpack.js.org/loaders/postcss-loader), [sass](https://webpack.js.org/loaders/sass-loader), and [less](https://webpack.js.org/loaders/less-loader) to name a few.

### 2.3 Loading Images

So now we're pulling in our CSS, but what about our images like backgrounds and icons? As of webpack 5, using the built-in [Asset Modules](https://webpack.js.org/guides/asset-modules/) we can easily incorporate those in our system as well:

**webpack.config.js**

```diff
 const path = require('path');

 module.exports = {
   entry: './src/index.js',
   output: {
     filename: 'bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
   module: {
     rules: [
       {
         test: /\.css$/i,
         use: ['style-loader', 'css-loader'],
       },
+      {
+        test: /\.(png|svg|jpg|jpeg|gif)$/i,
+        type: 'asset/resource',
+      },
     ],
   },
 };
```

Now, when you `import MyImage from './my-image.png'`, that image will be processed and added to your `output` directory *and* the `MyImage` variable will contain the final url of that image after processing. When using the [css-loader](https://webpack.js.org/loaders/css-loader), as shown above, a similar process will occur for `url('./my-image.png')` within your CSS. The loader will recognize this is a local file, and replace the `'./my-image.png'` path with the final path to the image in your `output` directory. The [html-loader](https://webpack.js.org/loaders/html-loader) handles `<img src="./my-image.png" />` in the same manner.

Let's add an image to our project and see how this works, you can use any image you like:

**project**

```diff
  webpack-demo
  |- package.json
  |- package-lock.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
+   |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```

**src/index.js**

```diff
 import _ from 'lodash';
 import './style.css';
+import Icon from './icon.png';

 function component() {
   const element = document.createElement('div');

   // Lodash, now imported by this script
   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
   element.classList.add('hello');

+  // Add the image to our existing div.
+  const myIcon = new Image();
+  myIcon.src = Icon;
+
+  element.appendChild(myIcon);
+
   return element;
 }

 document.body.appendChild(component());
```

**src/style.css**

```diff
 .hello {
   color: red;
+  background: url('./icon.png');
 }
```

Let's create a new build and open up the `index.html` file again:

```bash
$ npm run build

...
[webpack-cli] Compilation finished
assets by status 9.88 KiB [cached] 1 asset
asset bundle.js 73.4 KiB [emitted] [minimized] (name: main) 1 related asset
runtime modules 1.82 KiB 6 modules
orphan modules 326 bytes [orphan] 1 module
cacheable modules 540 KiB (javascript) 9.88 KiB (asset)
  modules by path ./node_modules/ 539 KiB
    modules by path ./node_modules/css-loader/dist/runtime/*.js 2.38 KiB
      ./node_modules/css-loader/dist/runtime/api.js 1.57 KiB [built] [code generated]
      ./node_modules/css-loader/dist/runtime/getUrl.js 830 bytes [built] [code generated]
    ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
    ./node_modules/style-loader/dist/runtime/injectStylesIntoStyleTag.js 6.67 KiB [built] [code generated]
  modules by path ./src/ 1.45 KiB (javascript) 9.88 KiB (asset)
    ./src/index.js + 1 modules 794 bytes [built] [code generated]
    ./src/icon.png 42 bytes (javascript) 9.88 KiB (asset) [built] [code generated]
    ./node_modules/css-loader/dist/cjs.js!./src/style.css 648 bytes [built] [code generated]
webpack 5.4.0 compiled successfully in 1972 ms
```

If all went well, you should now see your icon as a repeating background, as well as an `img` element beside our `Hello webpack` text. If you inspect this element, you'll see that the actual filename has changed to something like `29822eaa871e8eadeaa4.png`. This means webpack found our file in the `src` folder and processed it!

### 2.4 Loading Fonts

So what about other assets like fonts? The Asset Modules will take any file you load through them and output it to your build directory. This means we can use them for any kind of file, including fonts. Let's update our `webpack.config.js` to handle font files:

**webpack.config.js**

```diff
 const path = require('path');

 module.exports = {
   entry: './src/index.js',
   output: {
     filename: 'bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
   module: {
     rules: [
       {
         test: /\.css$/i,
         use: ['style-loader', 'css-loader'],
       },
       {
         test: /\.(png|svg|jpg|jpeg|gif)$/i,
         type: 'asset/resource',
       },
+      {
+        test: /\.(woff|woff2|eot|ttf|otf)$/i,
+        type: 'asset/resource',
+      },
     ],
   },
 };
```

Add some font files to your project:

**project**

```diff
  webpack-demo
  |- package.json
  |- package-lock.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
+   |- my-font.woff
+   |- my-font.woff2
    |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```

With the loader configured and fonts in place, you can incorporate them via an `@font-face` declaration. The local `url(...)` directive will be picked up by webpack, as it was with the image:

**src/style.css**

```diff
+@font-face {
+  font-family: 'MyFont';
+  src: url('./my-font.woff2') format('woff2'),
+    url('./my-font.woff') format('woff');
+  font-weight: 600;
+  font-style: normal;
+}
+
 .hello {
   color: red;
+  font-family: 'MyFont';
   background: url('./icon.png');
 }
```

Now run a new build and let's see if webpack handled our fonts:

```bash
$ npm run build

...
[webpack-cli] Compilation finished
assets by status 9.88 KiB [cached] 1 asset
assets by info 33.2 KiB [immutable]
  asset 55055dbfc7c6a83f60ba.woff 18.8 KiB [emitted] [immutable] [from: src/my-font.woff] (auxiliary name: main)
  asset 8f717b802eaab4d7fb94.woff2 14.5 KiB [emitted] [immutable] [from: src/my-font.woff2] (auxiliary name: main)
asset bundle.js 73.7 KiB [emitted] [minimized] (name: main) 1 related asset
runtime modules 1.82 KiB 6 modules
orphan modules 326 bytes [orphan] 1 module
cacheable modules 541 KiB (javascript) 43.1 KiB (asset)
  javascript modules 541 KiB
    modules by path ./node_modules/ 539 KiB
      modules by path ./node_modules/css-loader/dist/runtime/*.js 2.38 KiB 2 modules
      ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
      ./node_modules/style-loader/dist/runtime/injectStylesIntoStyleTag.js 6.67 KiB [built] [code generated]
    modules by path ./src/ 1.98 KiB
      ./src/index.js + 1 modules 794 bytes [built] [code generated]
      ./node_modules/css-loader/dist/cjs.js!./src/style.css 1.21 KiB [built] [code generated]
  asset modules 126 bytes (javascript) 43.1 KiB (asset)
    ./src/icon.png 42 bytes (javascript) 9.88 KiB (asset) [built] [code generated]
    ./src/my-font.woff2 42 bytes (javascript) 14.5 KiB (asset) [built] [code generated]
    ./src/my-font.woff 42 bytes (javascript) 18.8 KiB (asset) [built] [code generated]
webpack 5.4.0 compiled successfully in 2142 ms
```

Open up `dist/index.html` again and see if our `Hello webpack` text has changed to the new font. If all is well, you should see the changes.

### 2.5 Loading Data

Another useful asset that can be loaded is data, like JSON files, CSVs, TSVs, and XML. Support for JSON is actually built-in, similar to NodeJS, meaning `import Data from './data.json'` will work by default. To import CSVs, TSVs, and XML you could use the [csv-loader](https://github.com/theplatapi/csv-loader) and [xml-loader](https://github.com/gisikw/xml-loader). Let's handle loading all three:

```bash
npm install --save-dev csv-loader xml-loader
```

**webpack.config.js**

```diff
 const path = require('path');

 module.exports = {
   entry: './src/index.js',
   output: {
     filename: 'bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
   module: {
     rules: [
       {
         test: /\.css$/i,
         use: ['style-loader', 'css-loader'],
       },
       {
         test: /\.(png|svg|jpg|jpeg|gif)$/i,
         type: 'asset/resource',
       },
       {
         test: /\.(woff|woff2|eot|ttf|otf)$/i,
         type: 'asset/resource',
       },
+      {
+        test: /\.(csv|tsv)$/i,
+        use: ['csv-loader'],
+      },
+      {
+        test: /\.xml$/i,
+        use: ['xml-loader'],
+      },
     ],
   },
 };
```

Add some data files to your project:

**project**

```diff
  webpack-demo
  |- package.json
  |- package-lock.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
+   |- data.xml
+   |- data.csv
    |- my-font.woff
    |- my-font.woff2
    |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```

**src/data.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Mary</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Call Cindy on Tuesday</body>
</note>
```

**src/data.csv**

```csv
to,from,heading,body
Mary,John,Reminder,Call Cindy on Tuesday
Zoe,Bill,Reminder,Buy orange juice
Autumn,Lindsey,Letter,I miss you
```

Now you can `import` any one of those four types of data (JSON, CSV, TSV, XML) and the `Data` variable you import, will contain parsed JSON for consumption:

**src/index.js**

```diff
 import _ from 'lodash';
 import './style.css';
 import Icon from './icon.png';
+import Data from './data.xml';
+import Notes from './data.csv';

 function component() {
   const element = document.createElement('div');

   // Lodash, now imported by this script
   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
   element.classList.add('hello');

   // Add the image to our existing div.
   const myIcon = new Image();
   myIcon.src = Icon;

   element.appendChild(myIcon);

+  console.log(Data);
+  console.log(Notes);
+
   return element;
 }

 document.body.appendChild(component());
```

Re-run the `npm run build` command and open `dist/index.html`. If you look at the console in your developer tools, you should be able to see your imported data being logged to the console!

> :o:This can be especially helpful when implementing some sort of data visualization using a tool like [d3](https://github.com/d3). Instead of making an ajax request and parsing the data at runtime you can load it into your module during the build process so that the parsed data is ready to go as soon as the module hits the browser.
>

==:bangbang:Only the default export of JSON modules can be used without warning.==

```javascript
// No warning
import data from './data.json';
// Warning shown, this is not allowed by the spec.
import { foo } from './data.json';
```

#### 2.5.1 Customize parser of JSON modules

It's possible to import any `toml`, `yaml` or `json5` files as a JSON module by using a [custom parser](https://webpack.js.org/configuration/module/#ruleparserparse) instead of a specific webpack loader.

Let's say you have a `data.toml`, a `data.yaml` and a `data.json5` files under `src` folder:

**src/data.toml**

```toml
title = "TOML Example"

[owner]
name = "Tom Preston-Werner"
organization = "GitHub"
bio = "GitHub Cofounder & CEO\nLikes tater tots and beer."
dob = 1979-05-27T07:32:00Z
```

**src/data.yaml**

```yaml
title: YAML Example
owner:
  name: Tom Preston-Werner
  organization: GitHub
  bio: |-
    GitHub Cofounder & CEO
    Likes tater tots and beer.
  dob: 1979-05-27T07:32:00.000Z
```

**src/data.json5**

```json
{
  // comment
  title: 'JSON5 Example',
  owner: {
    name: 'Tom Preston-Werner',
    organization: 'GitHub',
    bio: 'GitHub Cofounder & CEO\n\
Likes tater tots and beer.',
    dob: '1979-05-27T07:32:00.000Z',
  },
}
```

Install `toml`, `yamljs` and `json5` packages first:

```bash
npm install toml yamljs json5 --save-dev
```

And configure them in your webpack configuration:

**webpack.config.js**

```diff
 const path = require('path');
+const toml = require('toml');
+const yaml = require('yamljs');
+const json5 = require('json5');

 module.exports = {
   entry: './src/index.js',
   output: {
     filename: 'bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
   module: {
     rules: [
       {
         test: /\.css$/i,
         use: ['style-loader', 'css-loader'],
       },
       {
         test: /\.(png|svg|jpg|jpeg|gif)$/i,
         type: 'asset/resource',
       },
       {
         test: /\.(woff|woff2|eot|ttf|otf)$/i,
         type: 'asset/resource',
       },
       {
         test: /\.(csv|tsv)$/i,
         use: ['csv-loader'],
       },
       {
         test: /\.xml$/i,
         use: ['xml-loader'],
       },
+      {
+        test: /\.toml$/i,
+        type: 'json',
+        parser: {
+          parse: toml.parse,
+        },
+      },
+      {
+        test: /\.yaml$/i,
+        type: 'json',
+        parser: {
+          parse: yaml.parse,
+        },
+      },
+      {
+        test: /\.json5$/i,
+        type: 'json',
+        parser: {
+          parse: json5.parse,
+        },
+      },
     ],
   },
 };
```

**src/index.js**

```diff
 import _ from 'lodash';
 import './style.css';
 import Icon from './icon.png';
 import Data from './data.xml';
 import Notes from './data.csv';
+import toml from './data.toml';
+import yaml from './data.yaml';
+import json from './data.json5';
+
+console.log(toml.title); // output `TOML Example`
+console.log(toml.owner.name); // output `Tom Preston-Werner`
+
+console.log(yaml.title); // output `YAML Example`
+console.log(yaml.owner.name); // output `Tom Preston-Werner`
+
+console.log(json.title); // output `JSON5 Example`
+console.log(json.owner.name); // output `Tom Preston-Werner`

 function component() {
   const element = document.createElement('div');

   // Lodash, now imported by this script
   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
   element.classList.add('hello');

   // Add the image to our existing div.
   const myIcon = new Image();
   myIcon.src = Icon;

   element.appendChild(myIcon);

   console.log(Data);
   console.log(Notes);

   return element;
 }

 document.body.appendChild(component());
```

Re-run the `npm run build` command and open `dist/index.html`. You should be able to see your imported data being logged to the console!

### 2.6 Global Assets

### 2.7 Wrapping up

## 3. Output Management

## 4. Development

